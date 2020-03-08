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
```
