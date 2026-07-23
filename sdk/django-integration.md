---
title: "Django Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/django-integration
canonical: https://vedika.io/docs/sdk/django-integration
---

# Django Integration

Full-stack astrology with Django

## Setup

```
# Create Django project
django-admin startproject astrology_project
cd astrology_project
python manage.py startapp charts

# Install SDK
pip install vedika-sdk django-environ
```

## Settings Configuration

```
# settings.py
import environ

env = environ.Env()
environ.Env.read_env()

VEDIKA_API_KEY = env('VEDIKA_API_KEY')

INSTALLED_APPS = [
    ...
    'rest_framework',
    'charts',
]
```

## Models

```
# charts/models.py
from django.db import models
from django.contrib.auth.models import User

class BirthProfile(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    name = models.CharField(max_length=100)
    datetime = models.DateTimeField()
    latitude = models.DecimalField(max_digits=9, decimal_places=6)
    longitude = models.DecimalField(max_digits=9, decimal_places=6)
    timezone = models.CharField(max_length=50, default='UTC')

    # Cached chart data
    sun_sign = models.CharField(max_length=20, blank=True)
    moon_sign = models.CharField(max_length=20, blank=True)
    ascendant = models.CharField(max_length=20, blank=True)
    chart_data = models.JSONField(null=True, blank=True)

    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f"{self.name} - {self.sun_sign}"
```

## Vedika Service

```
# charts/services.py
from django.conf import settings
from vedika import VedikaClient
from vedika.exceptions import VedikaError

class VedikaService:
    _client = None

    @classmethod
    def get_client(cls):
        if cls._client is None:
            cls._client = VedikaClient(api_key=settings.VEDIKA_API_KEY)
        return cls._client

    @classmethod
    def get_birth_chart(cls, birth_profile):
        client = cls.get_client()
        chart = client.birth_chart({
            'datetime': birth_profile.datetime.isoformat(),
            'latitude': float(birth_profile.latitude),
            'longitude': float(birth_profile.longitude)
        })
        return chart

    @classmethod
    def get_kundli_match(cls, profile1, profile2):
        client = cls.get_client()
        return client.kundli_match({
            'bride': {
                'datetime': profile1.datetime.isoformat(),
                'latitude': float(profile1.latitude),
                'longitude': float(profile1.longitude)
            },
            'groom': {
                'datetime': profile2.datetime.isoformat(),
                'latitude': float(profile2.latitude),
                'longitude': float(profile2.longitude)
            }
        })

    @classmethod
    def get_panchang(cls, date, latitude, longitude):
        client = cls.get_client()
        return client.panchang({
            'date': date,
            'latitude': latitude,
            'longitude': longitude
        })
```

## API Views (DRF)

```
# charts/views.py
from rest_framework import viewsets, status
from rest_framework.decorators import action
from rest_framework.response import Response
from .models import BirthProfile
from .serializers import BirthProfileSerializer, ChartSerializer
from .services import VedikaService

class BirthProfileViewSet(viewsets.ModelViewSet):
    serializer_class = BirthProfileSerializer

    def get_queryset(self):
        return BirthProfile.objects.filter(user=self.request.user)

    def perform_create(self, serializer):
        profile = serializer.save(user=self.request.user)
        # Generate chart on creation
        self._update_chart_data(profile)

    def _update_chart_data(self, profile):
        try:
            chart = VedikaService.get_birth_chart(profile)
            profile.sun_sign = chart.sun_sign
            profile.moon_sign = chart.moon_sign
            profile.ascendant = chart.ascendant
            profile.chart_data = {
                'planets': [
                    {'name': p.name, 'sign': p.sign, 'degree': p.degree}
                    for p in chart.planets
                ]
            }
            profile.save()
        except Exception as e:
            # Log error but don't fail
            pass

    @action(detail=True, methods=['get'])
    def chart(self, request, pk=None):
        profile = self.get_object()
        if not profile.chart_data:
            self._update_chart_data(profile)
        return Response({
            'sun_sign': profile.sun_sign,
            'moon_sign': profile.moon_sign,
            'ascendant': profile.ascendant,
            'planets': profile.chart_data.get('planets', [])
        })

    @action(detail=True, methods=['post'])
    def match(self, request, pk=None):
        profile = self.get_object()
        partner_id = request.data.get('partner_id')
        partner = BirthProfile.objects.get(id=partner_id, user=request.user)

        match = VedikaService.get_kundli_match(profile, partner)
        return Response({
            'score': match.total_score,
            'max_score': match.max_score,
            'percentage': match.percentage,
            'verdict': match.verdict
        })
```

## URL Configuration

```
# charts/urls.py
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from .views import BirthProfileViewSet

router = DefaultRouter()
router.register('profiles', BirthProfileViewSet, basename='profile')

urlpatterns = [
    path('api/', include(router.urls)),
]

# project/urls.py
from django.urls import path, include

urlpatterns = [
    path('', include('charts.urls')),
]
```

## Background Tasks (Celery)

```
# charts/tasks.py
from celery import shared_task
from .models import BirthProfile
from .services import VedikaService

@shared_task
def generate_chart_async(profile_id):
    """Generate chart in background."""
    profile = BirthProfile.objects.get(id=profile_id)
    chart = VedikaService.get_birth_chart(profile)

    profile.sun_sign = chart.sun_sign
    profile.moon_sign = chart.moon_sign
    profile.ascendant = chart.ascendant
    profile.chart_data = {'planets': [...]}
    profile.save()

@shared_task
def send_daily_horoscope():
    """Send daily horoscope to all users."""
    from datetime import date
    today = date.today().isoformat()

    profiles = BirthProfile.objects.all()
    for profile in profiles:
        # Generate personalized horoscope
        # Send email/notification
        pass
```

## Caching

```
# charts/views.py
from django.views.decorators.cache import cache_page
from django.utils.decorators import method_decorator

class PanchangView(APIView):
    @method_decorator(cache_page(60 * 60))  # Cache 1 hour
    def get(self, request):
        date = request.query_params.get('date')
        lat = float(request.query_params.get('lat', 28.6139))
        lng = float(request.query_params.get('lng', 77.2090))

        panchang = VedikaService.get_panchang(date, lat, lng)
        return Response({
            'tithi': panchang.tithi.name,
            'nakshatra': panchang.nakshatra.name
        })
```

## Admin Interface

```
# charts/admin.py
from django.contrib import admin
from .models import BirthProfile

@admin.register(BirthProfile)
class BirthProfileAdmin(admin.ModelAdmin):
    list_display = ['name', 'user', 'sun_sign', 'moon_sign', 'created_at']
    list_filter = ['sun_sign', 'moon_sign', 'created_at']
    search_fields = ['name', 'user__username']
    readonly_fields = ['sun_sign', 'moon_sign', 'ascendant', 'chart_data']
```

## Next Steps

[ **

#### Matching Portal

Build matrimonial sites

](https://vedika.io/docs/sdk/matching-portal) [ **

#### Async Django

Django Channels

](https://vedika.io/docs/sdk/python-async)

---
_Full page (always current): [https://vedika.io/docs/sdk/django-integration](https://vedika.io/docs/sdk/django-integration) · [Vedika docs](https://vedika.io/docs)_
