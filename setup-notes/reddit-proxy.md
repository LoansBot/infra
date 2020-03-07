# Reddit Proxy

The reddit-proxy listens on a particular queue from the AMQP server and proxies
the request to reddit. It sends logs to the database.

## Setup

Create a file `secrets.sh` which looks like the following (except filled in)

```bash
#!/usr/bin/env bash
export USER_AGENT="linux:com.redditloans:v2.0.0 (by /u/Tjstretchalot)"
export APPNAME=reddit-proxy
export PGHOST=10.0.0.47
export PGPORT=5432
export PGUSER=ec2-user
export PGPASSWORD=postgres
export PGDATABASE=postgres
export PYTHON_ARGS=-u
export AMQP_HOST=10.0.0.49
export AMQP_PORT=5672
export AMQP_USERNAME=guest
export AMQP_PASSWORD=guest
export AMQP_VHOST=/
export AMQP_QUEUE=rproxy
export MIN_TIME_BETWEEN_REQUESTS_S=2
export REDDIT_USERNAME=LoansBot
export REDDIT_PASSWORD=
export REDDIT_CLIENT_ID=
export REDDIT_CLIENT_SECRET=
```

```bash
sudo chmod +x secrets.sh
sudo chown root secrets.sh
sudo chgrp root secrets.sh
sudo yum -y install ruby
sudo yum -y install wget
wget https://aws-codedeploy-us-west-2.s3.us-west-2.amazonaws.com/latest/install
sudo chmod +x ./install
sudo ./install auto
sleep 1
sudo service codedeploy-agent status
```
