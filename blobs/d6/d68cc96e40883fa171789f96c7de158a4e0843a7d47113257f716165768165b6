// ── Clock & Weather Widget ──────────────────────────────────────────────
// Receives { hour, minute, weather } from the client and renders:
//   - An inline sun/moon icon with a 2D spin transition at dawn/dusk
//   - Moon phase is based on the hour — RDR3 cycles every night:
//     Full at dusk → shrinks to crescent at midnight → grows back to full at dawn
//   - A digital time readout with AM/PM
//   - A weather line with icon + text label (Rainy, Foggy, etc.)

(function () {
    'use strict';

    // ── Hour-based moon phase lookup ─────────────────────────────────
    // RDR3's moon visually cycles each night (same every night).
    // Mapped from in-game observation:
    //   8PM  Full  →  shrinks  →  12AM Crescent  →  grows  →  5AM Full
    // No new moon ever appears.
    var moonByHour = {
        20: '🌕',  // 8 PM  – Full Moon
        21: '🌔',  // 9 PM  – Waxing Gibbous  (shrinking)
        22: '🌖',  // 10 PM – Waning Gibbous
        23: '🌗',  // 11 PM – Last Quarter
         0: '🌒',  // 12 AM – Crescent
         1: '🌒',  // 1 AM  – Crescent
         2: '🌓',  // 2 AM  – First Quarter-ish
         3: '🌓',  // 3 AM  – First Quarter
         4: '🌔',  // 4 AM  – Waxing Gibbous  (growing)
         5: '🌕',  // 5 AM  – Full Moon
    };
    // Fallback for edge hours (6-7 AM before sun takes over, 18-19 sunset)
    var defaultMoon = '🌕';

    // ── Weather → emoji mapping ──────────────────────────────────────
    var weatherIcons = {
        sunny:          '☀️',
        highpressure:   '☀️',
        clear:          '☀️',
        extrasunny:     '☀️',
        clouds:         '⛅',
        misty:          '🌤️',
        overcast:       '☁️',
        overcastdark:   '☁️',
        fog:            '🌫️',
        drizzle:        '🌦️',
        shower:         '🌧️',
        rain:           '🌧️',
        thunder:        '⛈️',
        thunderstorm:   '⛈️',
        hurricane:      '🌀',
        hail:           '🌨️',
        snow:           '❄️',
        snowlight:      '🌨️',
        blizzard:       '🌬️',
        groundblizzard: '🌬️',
        whiteout:       '🌬️',
        sleet:          '🌨️',
        sandstorm:      '💨',
    };

    // ── Weather → human-readable label ───────────────────────────────
    var weatherLabels = {
        sunny:          'Sunny',
        highpressure:   'Clear',
        clear:          'Clear',
        extrasunny:     'Clear',
        clouds:         'Cloudy',
        misty:          'Misty',
        overcast:       'Overcast',
        overcastdark:   'Overcast',
        fog:            'Foggy',
        drizzle:        'Drizzle',
        shower:         'Showers',
        rain:           'Rainy',
        thunder:        'Thunder',
        thunderstorm:   'Thunderstorms',
        hurricane:      'Hurricane',
        hail:           'Hail',
        snow:           'Snowy',
        snowlight:      'Light Snow',
        blizzard:       'Blizzard',
        groundblizzard: 'Blizzard',
        whiteout:       'Whiteout',
        sleet:          'Sleet',
        sandstorm:      'Sandstorm',
    };

    var nightLabels = {
        sunny:        'Clear Night',
        highpressure: 'Clear Night',
        clear:        'Clear Night',
        extrasunny:   'Clear Night',
    };

    // ── DOM refs ─────────────────────────────────────────────────────
    var widget       = document.getElementById('clock-widget');
    var celestialEl  = document.getElementById('clock-celestial');
    var weatherIcon  = document.getElementById('clock-weather-icon');
    var weatherText  = document.getElementById('clock-weather-text');
    var timeText     = document.getElementById('clock-time-text');

    if (!widget) return;

    var lastWeather   = 'sunny';
    var wasNight      = null;   // null = first update, true/false after
    var spinTimer     = null;

    // ── Get moon emoji for the current hour ──────────────────────────
    function getMoonEmoji(hour) {
        return moonByHour[hour] || defaultMoon;
    }

    // ── Trigger a 2D spin, swapping the icon at the halfway point ────
    function spinTransition(newEmoji) {
        if (spinTimer) { clearTimeout(spinTimer); spinTimer = null; }

        celestialEl.classList.remove('spinning');
        // Force reflow so re-adding the class restarts the animation
        void celestialEl.offsetWidth;
        celestialEl.classList.add('spinning');

        // Swap content at the halfway mark (400ms of 800ms animation)
        spinTimer = setTimeout(function () {
            celestialEl.textContent = newEmoji;
        }, 400);

        // Clean up class when animation ends
        setTimeout(function () {
            celestialEl.classList.remove('spinning');
            spinTimer = null;
        }, 850);
    }

    function update(hour, minute, weather) {
        var h24 = hour + minute / 60;

        // ── Day/night determination ─────────────────────────────────
        var isNight = (h24 < 5.5 || h24 >= 20);

        // ── Celestial icon: spin when day↔night changes ─────────────
        var targetEmoji = isNight ? getMoonEmoji(hour) : '☀️';

        if (wasNight === null) {
            // First load — set immediately, no animation
            celestialEl.textContent = targetEmoji;
        } else if (isNight !== wasNight) {
            // Day/night changed — do a spin transition
            spinTransition(targetEmoji);
        } else if (isNight) {
            // Already night but moon phase might have changed
            celestialEl.textContent = targetEmoji;
        }
        wasNight = isNight;

        // ── Time-of-day class for colour theming ────────────────────
        widget.classList.remove('daytime', 'nighttime', 'sunrise', 'sunset');
        if (h24 >= 5.5 && h24 < 7)       widget.classList.add('sunrise');
        else if (h24 >= 7 && h24 < 18)    widget.classList.add('daytime');
        else if (h24 >= 18 && h24 < 20)   widget.classList.add('sunset');
        else                               widget.classList.add('nighttime');

        // ── Digital time ────────────────────────────────────────────
        var h12    = hour % 12 || 12;
        var ampm   = hour < 12 ? 'AM' : 'PM';
        var minStr = minute < 10 ? '0' + minute : '' + minute;
        timeText.textContent = h12 + ':' + minStr + ' ' + ampm;

        // ── Weather icon + label ────────────────────────────────────
        lastWeather = weather || lastWeather;
        var icon, label;
        if (isNight) {
            // At night show the moon phase as the weather-row icon for clear skies
            var isClear = !weatherIcons[lastWeather] || lastWeather === 'sunny' ||
                          lastWeather === 'clear' || lastWeather === 'highpressure' ||
                          lastWeather === 'extrasunny';
            icon  = isClear ? getMoonEmoji(hour) : (weatherIcons[lastWeather] || '☁️');
            label = nightLabels[lastWeather] || weatherLabels[lastWeather] || 'Clear Night';
        } else {
            icon  = weatherIcons[lastWeather] || weatherIcons.sunny;
            label = weatherLabels[lastWeather] || 'Clear';
        }
        weatherIcon.textContent = icon;
        weatherText.textContent = label;

        // Show the widget
        widget.style.display = 'flex';
    }

    // ── Listen for NUI messages ──────────────────────────────────────
    window.addEventListener('message', function (event) {
        if (event.data.type === 'updateClockWeather') {
            update(
                event.data.hour    || 0,
                event.data.minute  || 0,
                event.data.weather || null
            );
        }
    });
})();
