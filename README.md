api-proxy
=============

Cache requests to an external API using Nginx as a proxy server.

## Requirements
  - nginx

This has been run on Ubuntu 18.04 LTS using Nginx 1.14.0 with certbot (`python-certbot-nginx`), but probably works fine with other Linux distributions and Nginx version.

## Nginx config
Make a copy of the example configuration file:
  - `$ sudo cp api-proxy.example /etc/nginx/sites-available/api-proxy`

You'll need to adjust the following:
  - `server_name` (for both 80 and 443)
  - `ssl_certificate`, `ssl_certificate_key` and `ssl_trusted_certificate`
  - `api-gateway.external.com` (multiple instances)

You might also want to add a `proxy_set_header` with an User-Agent string at the very end of `location /`.

When done you should create a symlink, run Nginx's syntax test, and then restart Nginx:
  - `$ sudo ln -s /etc/nginx/sites-available/api-proxy /etc/nginx/sites-enabled/api-proxy`
  - `$ sudo nginx -t`
  - `$ sudo systemctl restart nginx`

## Statistics (optional)
Getting statistics for your caching is a good way of telling whether it works as intended, or if you need to do some adjustments.

Extend `/etc/nginx/nginx.conf` by adding an custom logging format:

```
http {
        ...
        log_format nginx_cache '$remote_addr - $upstream_cache_status [$time_local] '
                               '"$request" $status $body_bytes_sent '
                               '"$http_referer" "$http_user_agent" ';
```

Enable the format by uncommenting [line 18](api-proxy.example#L18). Crunch the log files using awk:
  - `awk '{print $3}' /var/log/nginx/cache.log | sort | uniq -c | sort -r`

You could get it emailed weekly by setting up a cron job:
  - `@weekly /bin/zcat --force /var/log/nginx/*-cache.log* | /usr/bin/awk '{print $3}' | sort | uniq -c | sort -r | mail -s "Caching statistics" $MAILTO -r $MAILFROM`

## Credits
  - Bram Neijt's «[Setting up Nginx caching for API use](https://bneijt.nl/blog/post/setting-up-nginx-caching-for-api-use/)» (July 20, 2013)
  - Juan Carlos's «[How to: Measure your NginX Cache Performance using $upstream_cache_status in a custom Cache log](https://kx.cloudingenium.com/technologies/web/nginx/measure-nginx-cache-performance-using-upstream_cache_status-custom-cache-log/)» (December 21, 2013)
  - Nginx [blog](https://www.nginx.com/blog/nginx-caching-guide/) and docs

## License
See the [LICENSE](LICENSE.md) file for license rights and limitations (MIT).
