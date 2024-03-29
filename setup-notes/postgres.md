# Postgres Setup

The postgres server acts a long-term persistent store of data, and is backed up
daily. We also run our migrations on the database server.

## Setup

```bash
sudo yum -y update
sudo amazon-linux-extras enable postgresql11
sudo yum clean metadata
sudo amazon-linux-extras install -y docker
sudo yum install -y postgresql libpq-devel
sudo service docker start
# NOTE -> we didn't configure log rotation when setting up the instance,
# so we have a cron job to truncate the log file. If setting up a new
# instance, it's better to add these flags: --log-driver local
# which will rotate the logs
sudo docker run --name postgres -e POSTGRES_USER='ec2-user' -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=postgres -d -p 5432:5432 postgres:11.5
```

Create a file `secrets.sh` which looks like the following (except filled in)

```bash
#!/usr/bin/env bash
export DATABASE_HOST=localhost
export DATABASE_PORT=5432
export DATABASE_USER=ec2-user
export DATABASE_PASSWORD=postgres
export DATABASE_DBNAME=postgres
export PGHOST=localhost
export PGPORT=5432
export PGUSER=ec2-user
export PGPASSWORD=postgres
export PGDATABASE=postgres
export AWS_ACCESS_KEY=
export AWS_SECRET_KEY=
export AWS_S3_BUCKET=
export AWS_S3_FOLDER=postgres/prod
```

Then resume setup

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
# Should get The AWS CodeDeploy agent is running
# if not, sudo service codedeploy-agent start
```

Then once the codedeploy agent installs the library

```bash
echo "#/usr/bin/env bash" > create_backup.sh
echo "cd /webapps/dbhelpers/src" >> create_backup.sh
echo ". /home/ec2-user/secrets.sh" >> create_backup.sh
echo "python3 create_backup.py" >> create_backup.sh
sudo chmod +x create_backup.sh
sudo chown root create_backup.sh
sudo chgrp root create_backup.sh

echo "#/usr/bin/env bash" > clean_expire_tables.sh
echo ". /home/ec2-user/secrets.sh" >> clean_expire_tables.sh
echo "psql -c \"DELETE FROM authtokens WHERE expires_at < NOW()\"" >> clean_expire_tables.sh
echo "psql -c \"DELETE FROM claim_tokens WHERE expires_at < NOW()\"" >> clean_expire_tables.sh
echo "psql -c \"DELETE FROM log_events WHERE created_at < NOW() - INTERVAL '7 days'\"" >> clean_expire_tables.sh
echo "psql -c \"DELETE FROM endpoint_users WHERE endpoint_id IN (SELECT endpoints.id FROM endpoints WHERE sunsets_on IS NOT NULL AND sunsets_on < CURRENT_DATE)\"" >> clean_expire_tables.sh
echo "psql -c \"DELETE FROM endpoint_alerts WHERE endpoint_id IN (SELECT endpoints.id FROM endpoints WHERE sunsets_on IS NOT NULL AND sunsets_on < CURRENT_DATE - INTERVAL '30 days')\"" >> clean_expire_tables.sh
sudo chmod +x clean_expire_tables.sh
sudo chown root clean_expire_tables.sh
sudo chgrp root clean_expire_tables.sh

# if we forgot log rotation
echo "#/usr/bin/env bash" > clean_logs.sh
echo "sudo truncate -s 0 $(sudo docker inspect --format='{{.LogPath}}' $(sudo docker ps | head -n 2 | tail -n 1 | cut -d" " -f1))" >> clean_logs.sh
chmod +x clean_logs.sh
chown root clean_logs.sh
chgrp root clean_logs.sh

echo "@reboot service docker start; docker start postgres" > dbcron
echo "@daily yum update -y; sleep 60; service docker start; docker start postgres" >> dbcron
echo "@daily /home/ec2-user/create_backup.sh" >> dbcron
echo "@hourly /home/ec2-user/clean_expire_tables.sh" >> dbcron
# if we forgot log rotation
echo "@monthly /home/ec2-user/clean_logs.sh" >> dbcron
sudo crontab dbcron
sudo rm dbcron
```
