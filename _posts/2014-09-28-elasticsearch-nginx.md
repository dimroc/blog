---
layout: post
title: Elasticsearch behind NGINX on AWS
---

Hosting on Heroku or any other multi-tenant PaaS means you're sharing IP security with other people. Now your web services require at least basic auth
when interacting with services like MySQL and Elasticsearch. We will run Elasticsearch behind nginx, which will enforce a username/pw for basic auth.

<!--more-->

To skip the details and get up and going locally, check out this [GitHub repo with a Vagrantfile that uses Docker images to run Elasticsearch behind nginx](https://github.com/dimroc/secure-elasticsearch-vagrant).

If you want a big boy's deployment for production, check out [AWS's OpsWorks Layer for Elasticsearch](http://blogs.aws.amazon.com/application-management/post/Tx3MEVKS0A4G7R5/Deploying-Elasticsearch-with-OpsWorks) 

On to the fun:

### High-level Overview

* Nginx will be the only public facing ip, and serve as the gateway to all Elasticsearch nodes.
* Nginx will enforce basic auth before proxying all HTTP requests to the ES nodes in a round robin fashion.
  * This will be done using an .htpasswd file.

After you've spun up Elasticsearch, go through the following steps:

### Example Nginx Proxy File

{% highlight apache %}

# Based on https://github.com/elasticsearch/kibana/blob/master/sample/nginx.conf

server {
  listen                80 default_server;

  location / {
    proxy_pass http://elasticsearch:9200; # elastic search node
    proxy_set_header Host      $host;
    proxy_set_header X-Real-IP $remote_addr;

    auth_basic "Restricted";
    auth_basic_user_file /etc/nginx/conf.d/elasticsearch.htpasswd;
  }
}
{% endhighlight %}

### Generate elasticsearch.htpasswd file

{% highlight bash %}

htpasswd -c nginx/elasticsearch.htpasswd elasticsearch

{% endhighlight %}

Now run nginx with it pointing to ES, and you're good to go.

Further reading: [Wattpad Engineering's Using Nginx as a proxy for Elasticsearch](http://engineering.wattpad.com/post/78037079531/using-nginx-as-a-proxy-for-elasticsearch-and-how-to).
