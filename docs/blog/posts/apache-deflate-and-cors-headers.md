---
authors:
  - copperlight
categories:
  - Apache
  - CORS
date: 2020-06-12
draft: false
pin: false
---

# Apache Deflate and CORS Headers

<div class="meta">
  <span class="discuss"><a class="github-button" href="https://github.com/copperlight/copperlight.github.io/issues" data-icon="octicon-issue-opened" aria-label="Discuss copperlight/copperlight.github.io on GitHub">Discuss</a></span>
</div><br/>

For internal static websites, you may want to configure CORS headers with generous permissions,
to improve the cross-site experience. A reasonable deflate configuration is provided which will
compress the largest elements of a website.

```
LoadModule deflate_module modules/mod_deflate.so

<IfModule mod_deflate.c>
  # Compress HTML, CSS, JavaScript, Text, XML and fonts
  AddOutputFilterByType DEFLATE application/javascript
  AddOutputFilterByType DEFLATE application/rss+xml
  AddOutputFilterByType DEFLATE application/vnd.ms-fontobject
  AddOutputFilterByType DEFLATE application/x-font
  AddOutputFilterByType DEFLATE application/x-font-opentype
  AddOutputFilterByType DEFLATE application/x-font-otf
  AddOutputFilterByType DEFLATE application/x-font-truetype
  AddOutputFilterByType DEFLATE application/x-font-ttf
  AddOutputFilterByType DEFLATE application/x-javascript
  AddOutputFilterByType DEFLATE application/xhtml+xml
  AddOutputFilterByType DEFLATE application/xml
  AddOutputFilterByType DEFLATE font/opentype
  AddOutputFilterByType DEFLATE font/otf
  AddOutputFilterByType DEFLATE font/ttf
  AddOutputFilterByType DEFLATE image/svg+xml
  AddOutputFilterByType DEFLATE image/x-icon
  AddOutputFilterByType DEFLATE text/css
  AddOutputFilterByType DEFLATE text/html
  AddOutputFilterByType DEFLATE text/javascript
  AddOutputFilterByType DEFLATE text/plain
  AddOutputFilterByType DEFLATE text/xml
</IfModule>

LoadModule headers_module modules/mod_headers.so

# Once a resource becomes stale, caches must not use their stale copy without successful
# validation on the origin server.
Header always set Cache-Control "public, must-revalidate, max-age=0"

# Expose the response to frontend JavaScript code, when the request's credentials mode is
# include. Credentials are cookies, authorization headers or TLS client certificates.
Header always set Access-Control-Allow-Credentials true

# When the Origin header is set, copy it from the request to the response.
SetEnvIf Origin "(.+)" HAVE_origin=1
RewriteCond %{HTTP:Origin} (.+)
RewriteRule .* - [E=ORIGIN:%1]
Header always set Access-Control-Allow-Origin "%{ORIGIN}e" env=HAVE_origin

# When the Access-Control-Allow-Methods header is set, replace it with GET,PATCH,POST,PUT,DELETE.
SetEnvIf Access-Control-Request-Method "(.+)" HAVE_method=1
Header always set Access-Control-Allow-Methods "GET,PATCH,POST,PUT,DELETE" env=HAVE_method

# When the Access-Control-Request-Headers header is set, copy it from the request to the response.
SetEnvIf Access-Control-Request-Headers "(.+)" HAVE_headers=1
RewriteCond %{HTTP:Access-Control-Request-Headers} (.+)
RewriteRule .* - [E=HEADERS:%1]
Header always set Access-Control-Allow-Headers "%{HEADERS}e" env=HAVE_headers
```
