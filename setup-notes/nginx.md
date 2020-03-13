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

```bash
sudo yum -y install ruby
sudo yum -y install wget
wget https://aws-codedeploy-us-west-2.s3.us-west-2.amazonaws.com/latest/install
sudo chmod +x ./install
sudo ./install auto
sleep 1
sudo service codedeploy-agent status
```
