# Memcached

The memcached server acts as a short-term non-persistent memmory store. Read
more [https://github.com/cache](here).

## Setup

We assume that the internal IP of the memcached server is 10.0.0.62

```bash
sudo yum -y update
sudo yum -y install libevent-devel
sudo yum -y install make glibc-devel gcc patch
mkdir cache
cd cache
wget https://memcached.org/latest
tar -zxf latest
cd memcached-1.5.22  # match whatever was extracted
./configure
make
sudo make install
cd ~
sudo /usr/local/bin/memcached -d -m 256 -u root -p 11211 -l 10.0.0.62
```

Testing (usually done via the bastion)

```bash
sudo yum -y install python3
sudo yum -y install git
sudo python3 -m pip install --upgrade pip
git clone --depth=1 https://github.com/LoansBot/cache.git
cd cache
python3 -m venv venv
. venv/bin/activate
python3 -m pip install --upgrade pip
python3 -m pip install -r requirements.txt
MEMCACHED_HOST=10.0.0.62 MEMCACHED_PORT=11211 python3 tests/test_connection.py
deactivate
```
