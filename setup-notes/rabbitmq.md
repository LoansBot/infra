# RabbitMQ

RabbitMQ acts as our advanced message queue and is the main way to facilitate
transient message transfer between services. Read more
[here](https://github.com/LoansBot/amqp).

## Setup

We assume rabbitmq is running on 10.0.0.49

```bash
sudo yum -y update
sudo amazon-linux-extras -y install docker
sudo usermod -a -G docker ec2-user
# REBOOT SERVER
sudo service docker start
docker info  # should work

docker pull rabbitmq

echo "service docker start" > init_rabbitmq
echo "docker container run --publish 5672:5672 --detach --name amqp rabbitmq" >> init_rabbitmq
chmod +x init_rabbitmq
sudo chown root init_rabbitmq
sudo chgrp root init_rabbitmq
sudo ./init_rabbitmq
sudo crontab -e  # @reboot /home/ec2-user/init_rabbitmq
```

Testing (usually done via the bastion)

```bash
sudo yum -y install python3
sudo yum -y install git
sudo python3 -m pip install --upgrade pip
git clone --depth=1 https://github.com/LoansBot/amqp.git
cd amqp
python3 -m venv venv
. venv/bin/activate
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
AMQP_HOST=10.0.0.49 AMQP_PORT=5672 AMQP_USERNAME=guest AMQP_PASSWORD=guest AMQP_VHOST=/ python3 tests/test_connection.py
deactivate
```
