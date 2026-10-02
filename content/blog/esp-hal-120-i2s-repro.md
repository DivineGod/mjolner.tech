+++
title = "esp-hal 1.2.0 i2s repro"
slug = "esp-hal-120-i2s-repro"
date = "2026-09-04T07:18:00+00:00"
include_in_feeds = true
discoverable = true
is_page = false
+++

Here are the result of trying to migrate from esp-hal 1.1.0 i2s (an unstable feature) to esp-hal 1.2.0 i2s.

Seeing as the feature is unstable, I wasn't surprised that I had some issues.

To document these and to aid the esp-hal team, I have created reproduction examples that demonstrate my approach for 1.1.0, and for upgrading to 1.2.0 which initially was based off the documentation suggesting the use of `wait_for_available_async()`, then ultimately finding a better solution.

[esp-hal 1.1.0 working solution](https://codeberg.org/voidedgin/esp-hal-i2s-repros/src/branch/hal-1.1.0)

[esp-hal 1.2.0 broken solution](https://codeberg.org/voidedgin/esp-hal-i2s-repros/src/branch/hal-1.2.0-broken)

[esp-hal 1.2.0 working solution](https://codeberg.org/voidedgin/esp-hal-i2s-repros/src/branch/hal-1.2.0-working)
