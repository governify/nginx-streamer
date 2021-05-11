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
 
There must be only the upstream related to the nginx active in the machine. Remove or comment unused lines.

Example of an Nginx working in a machine with Bluejay and Galibo deployed:
```nginx
(...)
stream {
    map $ssl_preread_server_name $name {
      ~^(.*)\.bluejay.governify.io bluejay;
      ~^(.*)\.galibo.governify.io galibo;
#      ~^(.*)\.osseco.governify.io osseco;
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