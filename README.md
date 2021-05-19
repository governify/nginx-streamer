# Nginx Streamer

Configured to work as a stream nginx to redirect to multiple Nginxs.

## Usage

There are only 2 configuration files in order for it to work:

### docker-compose.yml

Networks for each nginx have to be properly spelled. Only keep networks active in the machine or `docker-compose up -d` won't work. Remove or comment the lines.

Example of an Nginx working in a machine with Bluejay and Galibo deployed:
```yaml 
(...)
    networks:
      - governify-bluejay
      - governify-galibo
#      - governify-osseco
    mem_limit: 100m
    restart: 'on-failure:5'
networks:
  governify-bluejay:
    external: true
  governify-galibo:
    external: true
#  governify-osseco:
#    external: true
```


### nginx.conf
 
There must be only one http server and one upstream related to each active nginx in the machine. Remove or comment unused http servers and upstreams.

Example of an Nginx working in a machine with Bluejay and Galibo deployed:
```nginx
(...)
http {
  server {
    listen 80;
    server_name *.bluejay.governify.io;
    location / {
      proxy_pass       http://bluejay-nginx;
      proxy_set_header Host            $host;
      proxy_set_header X-Forwarded-For $remote_addr;
      }
  }

  server {
    listen 80;
    server_name *.galibo.governify.io;
    location / {
      proxy_pass       http://galibo-nginx;
      proxy_set_header Host            $host;
      proxy_set_header X-Forwarded-For $remote_addr;
      }
  }

#  server {
#    listen 80;
#    server_name *.osseco.governify.io;
#    location / {
#      proxy_pass       http://osseco-nginx;
#      proxy_set_header Host            $host;
#      proxy_set_header X-Forwarded-For $remote_addr;
#      }
#  }
#
#  server {
#    listen 80;
#    server_name *.governify.io;
#    location / {
#      proxy_pass       http://services-nginx;
#      proxy_set_header Host            $host;
#      proxy_set_header X-Forwarded-For $remote_addr;
#      }
#  }
}

# HTTPS streaming
stream {
    map $ssl_preread_server_name $name {
      ~^(.*)\.bluejay.governify.io bluejay;
      ~^(.*)\.galibo.governify.io galibo;
      ~^(.*)\.osseco.governify.io osseco;
      ~^(.*)\.governify.io services;
    }
    
    upstream bluejay {
      server bluejay-nginx:443;
    }

    upstream galibo {
      server galibo-nginx:443;
    }

#    upstream osseco {
#      server osseco-nginx:443;
#    }
#
#    upstream services {
#      server services-nginx:443;
#    } 
    
    server {
        listen 443;
        listen [::]:443;
        proxy_pass $name;   
        ssl_preread on;     
    }
}
```

### Deploy
Run `docker-compose up -d` and it will start streaming to the other nginxs. 

Make sure the **ports in the child Nginxs are not exposed**.