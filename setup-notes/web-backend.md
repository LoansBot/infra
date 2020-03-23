# Web Backend

The web backend serves API requests and may have many machine running the same
server (horizontal scaling). Each server may be running multiple processes;
uvicorn is launched within supervisor.

## Setup

Create a secrets file that follows the following format:

```bash
#!/usr/bin/env bash
export APPNAME=web-backend
export PGHOST=10.0.0.47
export PGPORT=5432
export PGDATABASE=postgres
export PGUSER=ec2-user
export PGPASSWORD=postgres
export AMQP_HOST=10.0.0.49
export AMQP_PORT=5672
export AMQP_USERNAME=guest
export AMQP_PASSWORD=guest
export AMQP_REDDIT_PROXY_QUEUE=rproxy
export AMQP_VHOST=/
export MEMCACHED_HOST=10.0.0.62
export MEMCACHED_PORT=11211
export WEB_CONCURRENCY=16
export WEBHOST=localhost
export WEBPORT=8000
export ROOT_DOMAIN=https://redditloans.com
export APP_VERSION_NUMBER=$(date --utc +%s)
export UVICORN_PATH=/usr/local/bin/uvicorn
export RATELIMIT_DISABLED=0
export HUMAN_PASSWORD_ITERS=1000000
export HCAPTCHA_SECRET_KEY=
export HCAPTCHA_DISABLED=0
```

```bash
sudo chmod +x secrets.sh
sudo chown root secrets.sh
sudo chgrp root secrets.sh
sudo yum -y update
sudo yum -y install ruby
sudo yum -y install wget
wget https://aws-codedeploy-us-west-2.s3.us-west-2.amazonaws.com/latest/install
sudo chmod +x ./install
sudo ./install auto
sleep 1
sudo service codedeploy-agent status
echo "@reboot /webapps/lbapi/after_install.sh ; /webapps/lbapi/application_start.sh" > cronjbs
echo "@daily yum update -y" >> cronjbs
sudo crontab cronjbs
rm cronjbs
```

## Complete setup script

```bash
#!/usr/bin/env bash
cd /home/ec2-user
echo '#!/usr/bin/env bash' > secrets.sh
echo "export APPNAME=web-backend" >> secrets.sh
echo "export PGHOST=10.0.0.47" >> secrets.sh
echo "export PGPORT=5432" >> secrets.sh
echo "export PGDATABASE=postgres" >> secrets.sh
echo "export PGUSER=ec2-user" >> secrets.sh
echo "export PGPASSWORD=postgres" >> secrets.sh
echo "export AMQP_HOST=10.0.0.49" >> secrets.sh
echo "export AMQP_PORT=5672" >> secrets.sh
echo "export AMQP_USERNAME=guest" >> secrets.sh
echo "export AMQP_PASSWORD=guest" >> secrets.sh
echo "export AMQP_REDDIT_PROXY_QUEUE=rproxy" >> secrets.sh
echo "export AMQP_VHOST=/" >> secrets.sh
echo "export MEMCACHED_HOST=10.0.0.62" >> secrets.sh
echo "export MEMCACHED_PORT=11211" >> secrets.sh
echo "export WEB_CONCURRENCY=16" >> secrets.sh
export WEBHOST=$(hostname -i | grep -Eo '10(\.[0-9]+){3}')
echo "export WEBHOST=$WEBHOST" >> secrets.sh
echo "export WEBPORT=8000" >> secrets.sh
echo "export ROOT_DOMAIN=https://redditloans.com" >> secrets.sh
echo "export APP_VERSION_NUMBER=$(date --utc +%s)" >> secrets.sh
echo "export UVICORN_PATH=/usr/local/bin/uvicorn" >> secrets.sh
echo "export RATELIMIT_DISABLED=0" >> secrets.sh
echo "export HUMAN_PASSWORD_ITERS=1000000" >> secrets.sh
echo "export HCAPTCHA_SECRET_KEY=" >> secrets.sh
echo "export HCAPTCHA_DISABLED=0" >> secrets.sh
sudo chmod +x secrets.sh
sudo chown root secrets.sh
sudo chgrp root secrets.sh
sudo yum -y update
sudo yum -y install ruby
sudo yum -y install wget
wget https://aws-codedeploy-us-west-2.s3.us-west-2.amazonaws.com/latest/install
sudo chmod +x ./install
sudo ./install auto
sleep 1
sudo service codedeploy-agent status
echo "@reboot /webapps/lbapi/after_install.sh ; /webapps/lbapi/application_start.sh" > cronjbs
echo "@daily yum update -y" >> cronjbs
sudo crontab cronjbs
rm cronjbs
```