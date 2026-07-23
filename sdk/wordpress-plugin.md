---
title: "WordPress Integration | Vedika SDK Documentation"
source: https://vedika.io/docs/sdk/wordpress-plugin
canonical: https://vedika.io/docs/sdk/wordpress-plugin
---

# WordPress Integration

Shortcodes, widgets, and Gutenberg blocks

## Plugin Setup

```
<?php
/**
 * Plugin Name: Vedika Astrology
 * Description: Add astrology features using Vedika API
 * Version: 1.0.0
 */

define('VEDIKA_PLUGIN_DIR', plugin_dir_path(__FILE__));

// Include the main class
require_once VEDIKA_PLUGIN_DIR . 'includes/class-vedika-api.php';
require_once VEDIKA_PLUGIN_DIR . 'includes/shortcodes.php';
require_once VEDIKA_PLUGIN_DIR . 'includes/widgets.php';

// Initialize
function vedika_init() {
    // Register settings
    register_setting('vedika_options', 'vedika_api_key');
}
add_action('admin_init', 'vedika_init');

// Admin menu
function vedika_admin_menu() {
    add_options_page(
        'Vedika Settings',
        'Vedika Astrology',
        'manage_options',
        'vedika-settings',
        'vedika_settings_page'
    );
}
add_action('admin_menu', 'vedika_admin_menu');
```

## API Wrapper Class

```
<?php
// includes/class-vedika-api.php

class Vedika_API {
    private $api_key;
    private $base_url = 'https://api.vedika.io';

    public function __construct() {
        $this->api_key = get_option('vedika_api_key');
    }

    public function get_birth_chart($datetime, $latitude, $longitude) {
        $response = wp_remote_post($this->base_url . '/v2/astrology/birth-chart', [
            'headers' => [
                'Content-Type' => 'application/json',
                'Authorization' => 'Bearer ' . $this->api_key
            ],
            'body' => json_encode([
                'datetime' => $datetime,
                'latitude' => $latitude,
                'longitude' => $longitude
            ])
        ]);

        if (is_wp_error($response)) {
            return null;
        }

        return json_decode(wp_remote_retrieve_body($response), true);
    }

    public function get_panchang($date, $latitude, $longitude) {
        $cache_key = 'vedika_panchang_' . md5($date . $latitude . $longitude);
        $cached = get_transient($cache_key);

        if ($cached !== false) {
            return $cached;
        }

        $response = wp_remote_post($this->base_url . '/v2/astrology/panchang', [
            'headers' => [
                'Content-Type' => 'application/json',
                'Authorization' => 'Bearer ' . $this->api_key
            ],
            'body' => json_encode([
                'date' => $date,
                'latitude' => $latitude,
                'longitude' => $longitude
            ])
        ]);

        if (!is_wp_error($response)) {
            $data = json_decode(wp_remote_retrieve_body($response), true);
            set_transient($cache_key, $data, HOUR_IN_SECONDS);
            return $data;
        }

        return null;
    }
}
```

## Shortcodes

```
<?php
// includes/shortcodes.php

// [vedika_panchang date="2026-01-07" lat="28.6139" lng="77.2090"]
function vedika_panchang_shortcode($atts) {
    $atts = shortcode_atts([
        'date' => date('Y-m-d'),
        'lat' => '28.6139',
        'lng' => '77.2090'
    ], $atts);

    $api = new Vedika_API();
    $panchang = $api->get_panchang($atts['date'], $atts['lat'], $atts['lng']);

    if (!$panchang) {
        return '<p>Unable to load panchang</p>';
    }

    ob_start();
    ?>
    <div class="vedika-panchang">
        <h3>Today's Panchang</h3>
        <ul>
            <li><strong>Tithi:</strong> <?php echo esc_html($panchang['tithi']['name']); ?></li>
            <li><strong>Nakshatra:</strong> <?php echo esc_html($panchang['nakshatra']['name']); ?></li>
            <li><strong>Yoga:</strong> <?php echo esc_html($panchang['yoga']['name']); ?></li>
            <li><strong>Karana:</strong> <?php echo esc_html($panchang['karana']['name']); ?></li>
        </ul>
    </div>
    <?php
    return ob_get_clean();
}
add_shortcode('vedika_panchang', 'vedika_panchang_shortcode');

// [vedika_horoscope sign="aries"]
function vedika_horoscope_shortcode($atts) {
    $atts = shortcode_atts([
        'sign' => 'aries'
    ], $atts);

    $api = new Vedika_API();
    $horoscope = $api->get_daily_horoscope($atts['sign']);

    if (!$horoscope) {
        return '<p>Unable to load horoscope</p>';
    }

    return sprintf(
        '<div class="vedika-horoscope"><h3>%s</h3><p>%s</p></div>',
        esc_html(ucfirst($atts['sign'])),
        esc_html($horoscope['prediction'])
    );
}
add_shortcode('vedika_horoscope', 'vedika_horoscope_shortcode');
```

## Panchang Widget

```
<?php
// includes/widgets.php

class Vedika_Panchang_Widget extends WP_Widget {
    public function __construct() {
        parent::__construct(
            'vedika_panchang_widget',
            'Vedika Panchang',
            ['description' => 'Display daily panchang']
        );
    }

    public function widget($args, $instance) {
        echo $args['before_widget'];
        echo $args['before_title'] . 'Today\'s Panchang' . $args['after_title'];

        $api = new Vedika_API();
        $panchang = $api->get_panchang(
            date('Y-m-d'),
            $instance['latitude'] ?? 28.6139,
            $instance['longitude'] ?? 77.2090
        );

        if ($panchang) {
            echo '<ul class="vedika-panchang-widget">';
            echo '<li>Tithi: ' . esc_html($panchang['tithi']['name']) . '</li>';
            echo '<li>Nakshatra: ' . esc_html($panchang['nakshatra']['name']) . '</li>';
            echo '</ul>';
        }

        echo $args['after_widget'];
    }

    public function form($instance) {
        $latitude = $instance['latitude'] ?? 28.6139;
        $longitude = $instance['longitude'] ?? 77.2090;
        ?>
        <p>
            <label>Latitude:</label>
            <input type="text" name="<?php echo $this->get_field_name('latitude'); ?>"
                   value="<?php echo esc_attr($latitude); ?>" />
        </p>
        <p>
            <label>Longitude:</label>
            <input type="text" name="<?php echo $this->get_field_name('longitude'); ?>"
                   value="<?php echo esc_attr($longitude); ?>" />
        </p>
        <?php
    }
}

function vedika_register_widgets() {
    register_widget('Vedika_Panchang_Widget');
}
add_action('widgets_init', 'vedika_register_widgets');
```

## Usage Examples

```
<!-- In posts or pages -->
[vedika_panchang]

[vedika_panchang date="2026-01-15" lat="19.0760" lng="72.8777"]

[vedika_horoscope sign="leo"]

<!-- Birth chart form -->
[vedika_chart_form]
```

## Settings Page

```
<?php
function vedika_settings_page() {
    if (!current_user_can('manage_options')) {
        return;
    }
    ?>
    <div class="wrap">
        <h1>Vedika Astrology Settings</h1>
        <form method="post" action="options.php">
            <?php settings_fields('vedika_options'); ?>
            <table class="form-table">
                <tr>
                    <th>API Key</th>
                    <td>
                        <input type="text" name="vedika_api_key"
                               value="<?php echo esc_attr(get_option('vedika_api_key')); ?>"
                               class="regular-text" />
                        <p class="description">
                            Get your API key from <a href="https://vedika.io/dashboard">vedika.io/dashboard</a>
                        </p>
                    </td>
                </tr>
            </table>
            <?php submit_button(); ?>
        </form>
    </div>
    <?php
}
```

## Next Steps

[ **

#### Shopify

E-commerce integration

](https://vedika.io/docs/sdk/shopify-integration) [ **

#### Panchang Widget

Embeddable calendar

](https://vedika.io/docs/sdk/panchang-widget)

---
_Full page (always current): [https://vedika.io/docs/sdk/wordpress-plugin](https://vedika.io/docs/sdk/wordpress-plugin) · [Vedika docs](https://vedika.io/docs)_
