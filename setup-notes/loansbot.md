# LoansBot

The LoansBot server requests and processes information from the reddit-proxy to
handle commands and other triggers which are posted on monitored reddit
servers.

## Setup

Create a file `secrets.sh` which looks like the following (except filled in)

```bash
#!/usr/bin/env bash
export APPNAME=loansbot
export AMQP_HOST=10.0.0.49
export AMQP_VHOST=/
export AMQP_PORT=5672
export AMQP_USERNAME=guest
export AMQP_PASSWORD=guest
export AMQP_REDDIT_PROXY_QUEUE=rproxy
export AMQP_RESPONSE_QUEUE_PREFIX=loansbot
export PGHOST=10.0.0.47
export PGPORT=5432
export PGUSER=ec2-user
export PGPASSWORD=postgres
export PGDATABASE=postgres
export SUBREDDITS=borrow,loansbot,borrowcommands
export MEMCACHED_HOST=10.0.0.62
export MEMCACHED_PORT=11211
export PYTHON_COMMAND=python3
export PYTHON_ARGS=-u
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
