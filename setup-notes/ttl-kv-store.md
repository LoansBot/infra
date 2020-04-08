# TTL Key-Value Store

A key value-store which allows setting expiration times on keys and stores
values on solid state drives.

## Purpose

One operation that nearly every application performs unsatisfactorily surrounds
high-cost moderate-volume computations which have a small to moderate payload
and are usually read and written infrequently. For example, the LoansBot needs
to check the karma of users as there is a karma restriction for interacting
with the LoansBot.

Common solutions are:

- Simply repeat the operation every time. This will typically result in the
  typical cost of a request being surprisingly high, for example, if the
  LoansBot did this then it takes 3 requests to respond to nearly any
  command instead of 1 (fetch user A, fetch user B, respond).
- Cache using a memory store like memcached or redis. This does reduce the
  number of API requests, but now the application is in a world that for every
  additional user we need another kilobyte of ram for a very long time. That
  can get expensive really quickly. In some cases a least-recently-used
  approach can help, but only if the particular question implies there are only
  a few users which are getting most of the benefit of caching, which is not
  always the case. For example, there might be 10 users which we see 30% of
  the benefits of caching from, but then it _rapidly_ falls of such that the
  next 30% of the benefit requires a hundred thousand users.
- Cache using columns on relevant tables on the database. This often solves the
  problem well, except now schemas become more than half, and in egregious
  cases, more than 90% columns which are only used in one spot of the code-
  base, where if you don't understand that particular algorithm used in that
  spot than it's not clear why the column is so important.
- Cache using a separate tagging system within the database. For example,
  a schema like `tags (id, name)` and `user_tags (user_id, tag_id)`. Or even
  `user_stuff (id, user_id, name, data)` where data is just a json blob. This
  idea is actually not dissimilar from caches, but it's definitely not using
  the _relational_ part of the database very well. Furthermore, once you want
  to add a TTL it can become a nightmare to fight the database tool to get the
  correct asymptotic complexity for this peculiar use-case.

Instead of any of the above solutions, this project simply acknowledges that
this is a pretty common problem for almost any project. Hence even if any
individual instance of this problem doesn't warrant additional infrastructure,
the _class_ of problems does.

The tool and infrastructure to solve this must have the following properties:

- Can store key/value pairs and lookup values by key. These problems are by
  nature used in a single spot for a very specific problem - so we can convert
  the problem into one or a few exact key match question relatively easily.
- The size of value-space should not significantly impact cost or performance
  within reason. In other words, we want to be paying around the cost for more
  SSDs, not the cost for more RAM. And we're willing to always perform around
  the speed of SSDs rather than the speed of RAM.
- There should be an easy way to set long but not infinite TTL on keys. Most of
  these problems, for example, get to safely assume a user isn't returning
  around a year later rather than a day.

## Setup

ArangoDB Community Edition (self-managed) on a single node.

```bash
sudo yum -y update
sudo yum install -y docker
sudo service docker start
sudo docker pull arangodb/arangodb

export ARANGOHOST=$(hostname -i | grep -Eo '10(\.[0-9]+){3}')
echo '#!/usr/bin/env bash' > secrets.sh
echo "export ARANGOHOST=$ARANGOHOST" >> secrets.sh
echo "export ARANGOPORT=8529" >> secrets.sh
sudo chmod +x secrets.sh
sudo chown root secrets.sh
sudo chgrp root secrets.sh

echo '#!/usr/bin/env bash' > start_arango.sh
echo "source secrets.sh" >> start_arango.sh
echo 'docker run -d -e ARANGO_NO_AUTH=1 -p $ARANGOHOST:$ARANGOPORT:8529 arangodb/arangodb arangod --server.endpoint tcp://0.0.0.0:8529' >> start_arango.sh
sudo chmod +x start_arango.sh
sudo chown root start_arango.sh
sudo chgrp root start_arango.sh

sudo ./start_arango.sh
```

```bash
echo "@daily yum -y update" > dbcron
echo "@reboot service docker start; sleep 5; /home/ec2-user/start_arango.sh" >> dbcron
sudo crontab dbcron
sudo rm dbcron
```