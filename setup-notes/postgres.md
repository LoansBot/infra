# Postgres Setup

The postgres server acts a long-term persistent store of data, and is backed up
daily. We also run our migrations on the database server.

## Setup

```bash
sudo yum -y update
sudo amazon-linux-extras install -y docker
sudo service docker start
sudo docker run --name postgres -e POSTGRES_USER='ec2-user' -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=postgres -d -p 5432:5432 postgres:12.2
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
sudo chmod +x clean_expire_tables.sh
sudo chown root clean_expire_tables.sh
sudo chgrp root clean_expire_tables.sh

echo "@reboot service docker start; docker start postgres" > dbcron
echo "0 0 * * * /home/ec2-user/create_backup.sh" >> dbcron
echo "0 * * * * /home/ec2-user/clean_expire_tables.sh" >> dbcron
sudo crontab dbcron
sudo rm dbcron
```
