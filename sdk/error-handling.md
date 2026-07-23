---
title: "Error Handling | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/error-handling
canonical: https://vedika.io/docs/sdk/error-handling
---

# Error Handling

Handle API errors gracefully in your application

## Basic Error Handling

```
import { VedikaClient, VedikaError } from '@vedika-io/sdk';

const vedika = new VedikaClient();

try {
  const chart = await vedika.birthChart({
    datetime: '1990-05-15T10:30:00+05:30',
    latitude: 28.6139,
    longitude: 77.2090
  });
} catch (error) {
  if (error instanceof VedikaError) {
    console.error('API Error:', error.message);
    console.error('Error Code:', error.code);
    console.error('Status:', error.status);
  } else {
    console.error('Unexpected error:', error);
  }
}
```

## Error Codes Reference

| Code | Status | Description | Action |
| --- | --- | --- | --- |
| `INVALID_API_KEY` | 401 | API key is invalid or missing | Check API key configuration |
| `EXPIRED_API_KEY` | 401 | API key has expired | Regenerate key in dashboard |
| `INSUFFICIENT_CREDITS` | 402 | Not enough credits for request | Add credits to account |
| `RATE_LIMITED` | 429 | Too many requests | Wait and retry with backoff |
| `INVALID_PARAMETERS` | 400 | Invalid request parameters | Check parameter values |
| `INVALID_DATE` | 400 | Invalid date format | Use ISO 8601 format |
| `INVALID_COORDINATES` | 400 | Latitude/longitude out of range | Lat: -90 to 90, Lng: -180 to 180 |
| `SERVICE_UNAVAILABLE` | 503 | Service temporarily unavailable | Retry with exponential backoff |
| `TIMEOUT` | 504 | Request timed out | Retry or increase timeout |

## VedikaError Object

```
interface VedikaError extends Error {
  code: string;           // Error code (e.g., 'INVALID_API_KEY')
  status: number;         // HTTP status code
  message: string;        // Human-readable message
  details?: object;       // Additional error details
  requestId?: string;     // Request ID for support
  retryable: boolean;     // Whether request can be retried
  retryAfter?: number;    // Seconds to wait before retry (for rate limits)
}

// Example error object
{
  name: 'VedikaError',
  code: 'RATE_LIMITED',
  status: 429,
  message: 'Rate limit exceeded. Please wait before retrying.',
  retryable: true,
  retryAfter: 60,
  requestId: 'req_abc123xyz'
}
```

## Handle Specific Errors

```
import { VedikaClient, VedikaError } from '@vedika-io/sdk';

async function fetchChart(birthDetails) {
  const vedika = new VedikaClient();

  try {
    return await vedika.birthChart(birthDetails);
  } catch (error) {
    if (!(error instanceof VedikaError)) {
      throw error;  // Re-throw unexpected errors
    }

    switch (error.code) {
      case 'INVALID_API_KEY':
      case 'EXPIRED_API_KEY':
        // Redirect to settings or show API key error
        console.error('Authentication failed. Please check your API key.');
        throw new Error('Please configure your API key');

      case 'INSUFFICIENT_CREDITS':
        // Redirect to billing or show upgrade prompt
        console.error('No credits remaining.');
        throw new Error('Please add credits to continue');

      case 'RATE_LIMITED':
        // Show rate limit message with retry time
        const waitTime = error.retryAfter || 60;
        console.error(`Rate limited. Retry in ${waitTime} seconds.`);
        throw new Error(`Too many requests. Please wait ${waitTime} seconds.`);

      case 'INVALID_DATE':
        // Show date format error
        console.error('Invalid date format:', error.details);
        throw new Error('Please enter a valid date (YYYY-MM-DD)');

      case 'INVALID_COORDINATES':
        // Show location error
        console.error('Invalid coordinates');
        throw new Error('Please enter valid location coordinates');

      case 'SERVICE_UNAVAILABLE':
      case 'TIMEOUT':
        // Show temporary error, maybe retry
        console.error('Service temporarily unavailable');
        throw new Error('Service is temporarily unavailable. Please try again.');

      default:
        console.error('Unexpected API error:', error.code);
        throw new Error('An unexpected error occurred. Please try again.');
    }
  }
}
```

## Automatic Retries

```
// SDK has built-in retry logic
const vedika = new VedikaClient({
  apiKey: process.env.VEDIKA_API_KEY,
  maxRetries: 3,           // Retry up to 3 times (default)
  retryDelay: 1000,        // Start with 1 second delay
  retryBackoff: 2,         // Double delay each retry
  timeout: 30000           // 30 second timeout
});

// Retries happen automatically for:
// - Network errors
// - 5xx server errors
// - 429 rate limit errors (respects Retry-After header)

// NOT retried:
// - 4xx client errors (invalid params, auth errors)
// - Timeout after configured duration
```

## Custom Retry Logic

```
async function withRetry(fn, options = {}) {
  const {
    maxRetries = 3,
    baseDelay = 1000,
    maxDelay = 30000,
    retryOn = ['RATE_LIMITED', 'SERVICE_UNAVAILABLE', 'TIMEOUT']
  } = options;

  let lastError;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      // Don't retry non-retryable errors
      if (error instanceof VedikaError && !retryOn.includes(error.code)) {
        throw error;
      }

      // Don't retry on last attempt
      if (attempt === maxRetries) {
        break;
      }

      // Calculate delay with exponential backoff
      let delay = baseDelay * Math.pow(2, attempt);

      // Respect Retry-After header for rate limits
      if (error.retryAfter) {
        delay = error.retryAfter * 1000;
      }

      // Cap the delay
      delay = Math.min(delay, maxDelay);

      console.log(`Retry ${attempt + 1}/${maxRetries} in ${delay}ms...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}

// Usage
const chart = await withRetry(
  () => vedika.birthChart(birthDetails),
  { maxRetries: 3, baseDelay: 2000 }
);
```

## Validate Before Request

```
import { validateBirthDetails } from '@vedika-io/sdk';

function validateInput(birthDetails) {
  const errors = [];

  // Date validation
  const date = new Date(birthDetails.datetime);
  if (isNaN(date.getTime())) {
    errors.push('Invalid date format. Use ISO 8601 (YYYY-MM-DDTHH:mm:ss+offset)');
  }

  // Latitude validation
  if (birthDetails.latitude < -90 || birthDetails.latitude > 90) {
    errors.push('Latitude must be between -90 and 90');
  }

  // Longitude validation
  if (birthDetails.longitude < -180 || birthDetails.longitude > 180) {
    errors.push('Longitude must be between -180 and 180');
  }

  // Future date check
  if (date > new Date()) {
    errors.push('Birth date cannot be in the future');
  }

  return errors;
}

// Usage
const errors = validateInput(birthDetails);
if (errors.length > 0) {
  console.error('Validation errors:', errors);
  return;
}

// Proceed with API call
const chart = await vedika.birthChart(birthDetails);
```

## Global Error Handler

```
// Create a wrapper with global error handling
class VedikaService {
  constructor(apiKey) {
    this.client = new VedikaClient({ apiKey });
    this.onError = null;  // Error callback
  }

  setErrorHandler(handler) {
    this.onError = handler;
  }

  async execute(method, params) {
    try {
      return await this.client[method](params);
    } catch (error) {
      // Log error
      console.error(`Vedika API Error [${method}]:`, error);

      // Track error (e.g., Sentry)
      if (typeof Sentry !== 'undefined') {
        Sentry.captureException(error, {
          tags: { api: 'vedika', method },
          extra: { params }
        });
      }

      // Call error handler
      if (this.onError) {
        this.onError(error, method, params);
      }

      throw error;
    }
  }

  birthChart(params) {
    return this.execute('birthChart', params);
  }

  panchang(params) {
    return this.execute('panchang', params);
  }

  // ... other methods
}

// Usage
const vedika = new VedikaService(process.env.VEDIKA_API_KEY);

vedika.setErrorHandler((error, method, params) => {
  // Show user-friendly toast notification
  showToast({
    type: 'error',
    message: getErrorMessage(error)
  });
});

function getErrorMessage(error) {
  const messages = {
    'INSUFFICIENT_CREDITS': 'Please add credits to continue using the service.',
    'RATE_LIMITED': 'Too many requests. Please slow down.',
    'SERVICE_UNAVAILABLE': 'Service temporarily unavailable. Please try again.'
  };
  return messages[error.code] || 'An error occurred. Please try again.';
}
```

## Related Topics

[ **

#### Authentication

API key setup

](https://vedika.io/docs/sdk/authentication) [ **

#### Streaming

Handle stream errors

](https://vedika.io/docs/sdk/streaming) [ **

#### API Status

Check service health

](https://vedika.io/status)

---
_Full page (always current): [https://vedika.io/docs/sdk/error-handling](https://vedika.io/docs/sdk/error-handling) · [Vedika docs](https://vedika.io/docs)_
