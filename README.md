#### obnginx
#####4 Reverse proxy
######Config a basic
set long lived HTTP cache headers
```
location /assets{
  expires max;  (10 year) or 10d  (10 days)
  add_header Cache-Control public;
}
```
test
```
curl -I 127.0.0.1/assets/a.jpg
```
