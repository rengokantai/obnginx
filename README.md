#### obnginx
#####1 Getting Started
######Install from source
```
tar -zxvf nginx-1.9.2.tar.gz
./configure
make
make install
```
On ubuntu machine, must install
```
apt-get install libpcre3-dev zlib1g-dev libssl-dev
```
By default the path should be
```
/usr/local/nginx/sbin/nginx -V
```
change path:
```
./configure --prefix=/usr --conf-path=/etc/nginx
```
configure with other modules
```
./configure --with-http_ssl_module --with-http_spdy_module...
```
#####2 Basic Configuration
If nginx was installed from source, the path to conf file should be
```
/usr/local/nginx/conf
```
If from package,then the location might be
```
/etc/nginx
/usr/local/etc/nginx
```
/etc/n
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

######more rebust
```
http{
  upstream rails{
    server 127.0.0.0:3000;
  }
  server{
    listen *:80;
    root /public;
  
    location / {
      try_files $uri $uri/index.html @rails;
    }
  
    location @rails{
      proxy_pass http://rails
    }
  }
}
```
######custom error page
hide nginx version
```
server_tokens off;
```
override default error pages
```
location / {
  error_page 404 /404.html
  error_page 500 501 502 503 /50x.html
}
```
Or, redirect  
change status code
```
error_page 404 =200 /index.html
```
redirect another location block
```
error_page 404 http://google.com
```

redirect to a named location block
```
location /{
  error_page 404 = @fallback;
}
location @fallback{
  proxy_pass http://
}
```

setting X-Forwardeed-Proto (add a new header)
```
location @fallback{
  proxy_set_header X-Forward-Proto $scheme;
  proxy_pass http://
}
```
sometimes override the host header with proxy_set_header
```
proxy_set_header Host $host
```
Real IP moudle
```
http{
  real_ip_header X-Real-IP;            //Or  proxy_set_header X-Real_IP $remote_addr;
  set_real_ip_from 1.2.2.4;   //iddress of load balancer
  set_real_ip_from 1.2.3.4/24; 
  server{
  }
}
```
######Websockets
######Future

#####5 Load Balancing
weighted servers
```
upstream a{
  server 1 weight=1;
  server 2 weight=2;
}
```
health checks
```
upstream a{
  server 1 max_fails=1 fail_timeout=10;
  server 2 max_fails=100;
}
```
Or set 502 503 504 in upstream (do not set 500, it might be appplication code error
```
location /{
  proxy_next_upstream http_502 http_503 http_504;
  proxy_pass http://xx;
}
```

######mark as down(temp)
```
upstream x{
  server  a.com down;
}
```
######mark as backup
```
upstream x{
  server  a.com backup;
}
```
######slow start
######Load Balancing Methods
```
upstream a {
  least_conn; // OR ip_hash;
}
```
base on arbitrary variable
```
upstream a {
  hash $remote_addr/$uri consistent;
}
```
######C10K with nginx (serving 10000 concurrrent connections)
