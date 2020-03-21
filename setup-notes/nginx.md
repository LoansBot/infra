# Nginx

The nginx server acts as a reverse proxy for the api endpoints by routing
them to the web-backend package. Additionally, it stores static files for
the website (and correctly serves 304 responses to allow client-side disk
caching).

## Setup

```bash
sudo yum -y update
sudo amazon-linux-extras install -y nginx1=stable
sudo service nginx start
sudo vim /etc/nginx/nginx.conf  # configure proxy_pass for /api/
sudo service nginx restart
echo "@reboot service nginx start" | sudo crontab -
```

Configure caching and proxy in /etc/nginx/nginx.conf. Suggested
format is something like the following, which will set a 1 day
cache and disable the pre-HTTP/1.1 style caching.

```conf
        location ^~ /api/ {
            proxy_pass http://10.0.0.38:8000/;
        }

        location / {
            etag off;
            if_modified_since off;
            add_header Last-Modified "";
            add_header Cache-Control "public, max-age=86400, stale-while-revalidate=604800, stale-if-error=604800";
        }
```

And reload the service

```bash
sudo nginx -t && sudo service nginx reload
```

```bash
sudo yum -y install ruby
sudo yum -y install wget
wget https://aws-codedeploy-us-west-2.s3.us-west-2.amazonaws.com/latest/install
sudo chmod +x ./install
sudo ./install auto
sleep 1
sudo service codedeploy-agent status
echo "@daily yum update -y" > cronjbs
sudo crontab cronjbs
rm cronjbs
```

```bash
wget https://dl.eff.org/certbot-auto
chmod a+x certbot-auto
sudo ./certbot-auto --nginx -d staging.redditloans.com
```