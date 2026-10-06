# Vinyl for Drupal on Wodby

What this service adds to the Vinyl service it is based on.

## Drupal preset

`VARNISH_CONFIG_PRESET` is set to `drupal`, so the image renders its Drupal rules into `/etc/varnish/preset.vcl`:

- Requests matching `VARNISH_DRUPAL_EXCLUDE_URLS` go to the backend uncached. The default covers `/update.php`, `/admin`, `/system/files/`, `/flag/` and AJAX paths, with an optional two-letter language prefix. Batch pages are piped.
- Only cookies matching `VARNISH_DRUPAL_PRESERVED_COOKIES` are kept (by default Drupal session cookies and `NO_CACHE`); all others are removed before the cache lookup. A request that still has a cookie is not cached, so pages of signed-in users always come from Drupal. `VARNISH_KEEP_ALL_COOKIES` does not change this.

The preset template is the `drupal` config of the manifest (`/etc/gotpl/presets/drupal.vcl.tmpl`). Override it there, not in the rendered file.

## Cache tags

- `VARNISHD_PARAM_HTTP_RESP_HDR_LEN` is raised so that Drupal's long cache-tag headers fit.
- A `BAN` request with a `Cache-Tags` header invalidates every cached object whose `Cache-Tags` response header matches it. For tag-based invalidation Drupal must send its tags in a `Cache-Tags` response header and send its invalidations as `BAN` requests to this service on port `6081`. That is done with Drupal's purge modules and configuration; this service does not configure Drupal.

## Check the result

- An anonymous page requested twice returns `X-VC-Cache: HIT` the second time. A `MISS` every time usually means the response sets a cookie or sends `Cache-Control: private` or `no-cache`: Drupal's page cache maximum age must be above zero.
- The reason is in `X-VC-Cacheable`, delivered only when the backend response carries the header `X-VC-Debug: true`.
