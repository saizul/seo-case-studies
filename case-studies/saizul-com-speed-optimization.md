# Case study: speeding up saizul.com (WordPress + Elementor)

Date: November 2026
Site: [saizul.com](https://saizul.com), the author's own site

## Context

saizul.com runs WordPress with Elementor on an Ubuntu VPS managed with CloudPanel. The stack is Nginx, PHP 8.3, Redis for object caching and the Cache Enabler plugin for page caching. The server is in the EU, while most visitors are in Bangladesh.

## Problem

Pages felt slow, and uncached requests were slow on the server itself. Two things needed answers: how much of the delay came from the server and the WordPress setup, and how much came from the distance between the visitor and the server.

## Diagnosis method

The work separated server time from network time:

- PHP boot time of WordPress without page cache was timed from the command line.
- Local response time for the home page was measured on the server itself, which removes the network from the picture.
- Full page load was measured in a test browser, which includes everything a visitor's browser has to fetch.

Comparing the three showed where time was spent. Uncached PHP boot was slow, the portfolio images were heavier than they needed to be, and once the server side was fast the remaining delay was network distance.

## Actions

1. **Deactivated an unused WooCommerce plugin.** It was not used on the site but added about 2 seconds to every uncached request. A database backup was taken first, then the plugin was deactivated.
2. **Converted the portfolio and home screenshot images to WebP at 480 px wide.** Each image went from roughly 130 to 150 KB down to roughly 7 to 19 KB.
3. **Cached WordPress REST API responses for the posts endpoint for 10 minutes** with a small must-use plugin (mu-plugin), so repeated requests do not rebuild the same response.
4. **Redesigned the 404 page**, so visitors who land on a missing URL see a useful page.
5. **Applied PHP security patching across the server.**

## Results

| Measure | Before | After |
|---------|--------|-------|
| Full page load in a test browser | about 9.4 s | about 6.4 s |
| Uncached PHP boot (command-line timing) | about 7 s | about 3 s |
| Server-side local response, home page | not recorded | about 0.4 to 1.0 s |
| Portfolio and home screenshot images | about 130 to 150 KB each | about 7 to 19 KB each (about 88% smaller) |

The unused plugin alone added about 2 s to each uncached request.

## What is next

The server side is now fast: a local home page response takes about 0.4 to 1.0 s, yet a full page load in the test browser still takes about 6.4 s. The remaining bottleneck is network distance between visitors in Bangladesh and a server in the EU. The recommended next step is to put a CDN in front of the site, for example Cloudflare, so that static assets and cached pages are served from a location closer to visitors.

## Lessons

- Remove what you do not use. A single unused plugin cost about 2 s on every uncached request.
- Measure on the server and in the browser separately. Server-side numbers show what code and configuration can fix; the gap to the browser number shows what only the network path can fix.
- Image weight adds up. Resizing screenshots to the size they are displayed at and serving WebP cut each image by about 88%.
- Take a backup before deactivating or removing anything on a live site.
- Cache cheap, repeated work such as REST responses, and keep the cache lifetime short enough (10 minutes here) that content stays fresh.
- When the server is no longer the bottleneck, more server tuning has little left to give. Geography is the next lever.
