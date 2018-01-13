title: Using NGINX As Reverse Proxy
comments: true
date: 2018-01-13 18:19:49
tags:
    - nginx
    - reverse-proxy
categories: Web
---
{% asset_img nginxhero.jpg %}
### Use Case
When hosting our web applications, we often have one public IP address (i.e., an IP address visible to the outside world) using which we want to host multiple web apps. For example, one may wants to host three different web apps respectively for example1.com, example2.com, and example1.com/images on the same machine using a single IP address.

### How can we do that? 
Well, the good news is Internet browsers send the domain name inside HTTP requests and all we need to do is to parse the requested domain name and URL and then route the HTTP request to the actual web server. This can be easily by done with nginx by configuring it as a reverse proxy for different applications.

### Intall nginx (Ubuntu)
Install nginx using apt-get:
```
$ sudo apt-get install nginx -y
```

### Configuration
nginx default config file is located at `/etc/nginx/sites-enabled/default`, this is the file we need to modify.

Let say we want to configure nginx to route requests from /blog, and /mail, respectively onto localhost:3000 and localhost:3001.

```
                  +--- host/blog --------> node.js on localhost:3000
                  |
users --> nginx --|
                  |
                  +--- host/mail ---> node.js on localhost:3001
```

Lets modify the default nginx configuration (`/etc/nginx/sites-enabled/default`) to achieve the above use case as shown below

```nginx
    location / {
            # First attempt to serve request as file, then
            # as directory, then fall back to displaying a 404.
            try_files $uri $uri/ =404;
    }

    location /blog {
            rewrite ^/blog(.*) /$1 break;
            proxy_pass http://127.0.0.1:3000;
    }

    location /mail {
            rewrite ^/mail(.*) /$1 break;
            proxy_pass http://127.0.0.1:3001;
    }
```
`proxy_pass` simply tells `nginx` to forward requests to /blog to the server listening on http://127.0.0.1:3000.
`rewrite ^/blog(.*) /$1 break;` regular expression tells `nginx` to capture anything after `/blog` and pass it to the proxy pass, this is needed because our nodejs web servers are listening on `/` not on `/blog`. `break` tells `nginx` to stop further lookup.

### Reload NGINX
After making the above configuration changes, we need to reload `nginx` (no need to restart)
```
sudo service nginx reload
```
Once it is sucessfully reloaded, we can test the above configuration by making requests to the above routes in our browser, like

`http://YOUR_PUBLIC_IP/blog` and `http://YOUR_PUBLIC_IP/mail` should serve the response from nodejs applications running on port 3000 and 3001 respectively. Voila!!

Here is a sample expressjs app, if you want to play with https://gist.github.com/ibhi/0ab0342ae140612078fdd3f7e1cdf6f1

Please let me know your feedback or comments in the comments section below.
