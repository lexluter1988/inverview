
### initial data from https://hackattic.com/challenges/dockerized_solutions

```
https://hackattic.com/challenges/dockerized_solutions/problem?access_token=96fc810308712ece

{"credentials": {"user": "autumn-credit@hackattic.com", "password": "M8SAK0R31Z"}, "ignition_key": "WE4W04T36WTEAEEFNRU6KQIL276A", "trigger_token": "4822acf5.7341.4c19.8f7c.8d33670aae1c"}
```


#### created vps on yandex and domain on reg.ru

```
84.201.166.224 ( added A records and yandex ns as primary for domain )

www.lexxsmith.online. ( also created let's enctrypt for it )
```


#### configured docker registry there and auth for that user with that password ( basically by Digital Ocean guide )

```
$ cat /etc/nginx/sites-available/www.lexxsmith.online 
server {

        root /var/www/lexxsmith.online/html;
        index index.html index.htm index.nginx-debian.html;

        server_name  www.lexxsmith.online;

        #location / {
        #        try_files $uri $uri/ =404;
        #}
location / {
    # Do not allow connections from docker 1.5 and earlier
    # docker pre-1.6.0 did not properly set the user agent on ping, catch "Go *" user agents
    if ($http_user_agent ~ "^(docker\/1\.(3|4|5(?!\.[0-9]-dev))|Go ).*$" ) {
      return 404;
    }

    proxy_pass                          http://localhost:5000;
    proxy_set_header  Host              $http_host;   # required for docker client's sake
    proxy_set_header  X-Real-IP         $remote_addr; # pass on real client's IP
    proxy_set_header  X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header  X-Forwarded-Proto $scheme;
    proxy_read_timeout                  900;
}

    listen [::]:443 ssl ipv6only=on; # managed by Certbot
    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/www.lexxsmith.online/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/www.lexxsmith.online/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot

}
server {
    if ($host = www.lexxsmith.online) {
        return 301 https://$host$request_uri;
    } # managed by Certbot


        listen 80;
        listen [::]:80;

        server_name  www.lexxsmith.online;
    return 404; # managed by Certbot

```

#### So, you can do this from any machine

```
$ docker login https://www.lexxsmith.online
# push and pull
```

