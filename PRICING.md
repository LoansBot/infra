# Pricing

This file explains which actual instances are used to host each of the services
right now, and the corresponding prices. There are other small costs not related
to the instances.

## Motivation

These service sizes, at the time of writing, are chosen by intuition only. If
there are significant bottlenecks they will be addressed when measured. These
instances tend to err on the smaller side, but we try to choose the correct
instance type (bursting vs memory-optimized, etc) to start since it does take
a some amount of effort to swap instances out, even with 98% of the work being
done by AWS.

- The S3 server is not simple to optimize and is heavily dependent on usage, so
  it's worth waiting to verify before attempting anything clever here.
- The Nginx server needs to serve a fairly light but consistent load caused by
  bots (estimate: 1 req/sec). Further, a lighter but more centralized load from
  users clicking through from reddit for mobile queries (likely 0-3 req/s).
  There is a negligible load on the nginx server from moderators and power
  users. Finally, there are occassionally people either port scanning or
  not correctly setting their back-off settings that can cause short term load
  spikes (~100 req/sec).
- The load on the web-worker is expected to be similar to the Nginx server,
  except the power users and moderators cost so much more per request that they
  are likely to dominate the load, so the load is signficantly burstier.
- The memcached server is likely to barely use any CPU at this usage, and it
  doesn't need more than a few hundred megabytes of memory most likely. The
  load will mainly be from the LoansBot, which will be very steady, and some
  bursting from the web worker is possible.
- The postgres server will not behave satisfactorily unless the whole database
  is in memory (likely multiple times over for indexes). Furthermore, it has by
  far the highest CPU requirements of any service due to servicing all the
  complex joins and queries that are used for everything from moderators to
  analytics. It also has a reasonably high steady and burst load from logging and backups.
- The RabbitMQ server is going to have a very steady low load from the LoansBot
  and a negligible load from the website.
- The LoansBot is expected to have a nearly negligible load.
- The reddit proxy will have a negligible CPU and memory load, as it only
  processes at the reddit API limit.

## Details

All instances are purchased on a 3 year no-upfront convertible contract. S3
prices are based on usage All services run on the default AWS linux AMI.

All numbers are derived from
[the simple monthly calculator](https://calculator.s3.amazonaws.com/index.html).

Each instance has an associated minimum EBS volume of 8gb at $0.10/gb/mo

- Nginx: `t3a.micro` (2 vCPU @ 10%, 1gb mem) at $3.44/mo + $0.80/mo
- Web worker: 1 `t3a.small` (2 vCPU @ 20%, 2gb mem) running 16 actual workers at $6.87/mo + $0.80/mo
- Memcached: `t3a.nano` (2 vCPU @ 5%, 512mb mem) at $1.68/mo + $0.80/mo
- Postgres: `m5a.large` (2 vCPU @ 100%, 8gb mem) at $31.39/mo + $0.80/mo
- RabbitMQ: `t3a.nano` (2 vCPU @ 5%, 512mb mem) at $1.68/mo + $0.80/mo
- Reddit-Proxy: `t3a.nano` (2 vCPU @ 5%, 512mb mem) at $1.68/mo + $0.80/mo
- LoansBot: `t3a.nano` (2 vCPU @ 5%, 512mb mem) at $1.68/mo + $0.80/mo
- NAT Instance: `t3a.nano` (2 vCPU @ 5%, 512mb mem) at $1.68/mo + $0.80/mo
- TTL Key-Value: `t3a.micro` (2 vCPU @ 10%, 1gb mem) at $3.44/mo + $0.80/mo

Transfer estimates:

For bots we assume people are using the worst possible format (uncondensed
json) and fetching the current limit of 99 loans (although this limit will be
raised).

1 req/s * 2.63m s/mo = 2.63m req/mo
2.63m req/mo * 26kb/req = 68 gb/mo

For people we assume they are using the best possible format (since we do),
and are still fetching 99 loans: 6kb. Further we assume there are 4 API
calls per web page, so 0.8 req/s.

0.8 req/s * 2.63m s/mo = 2.1m req/mo
2.1m req/mo * 6kb/req = 12.6 gb/mo

Total transfer out: 4.73m req/mo, 80.6gb/mo

80.6gb/mo * $0.09/gb = $7.25/mo

S3 price estimates:

Currently the bulkiest page is the old query page at 6kb, whereas the most used
page is the mobile query at 1.1kb. Since there won't be any server-side
rendering anymore, it'll be more than the mobile query page. These calculations
assume 3kb/req is uncached.

The bot load will go entirely through the EC2 instances and is accounted for
above, so let's assume an average of 0.2 requests/second. That's 526k req/mo
and about 1.58 gb/mo.

There will also be roughly 46gb stored in S3 from backups (assuming they are
stored at 128mb/ea and last 365 days), plus a small amount from miscellaneous
caching and the actual html/css/json.

- 526k GET requests per month * $0.0004 / 1,000 GET requests = $0.21/mo
- 1.56 gb/mo: first gb is free, so 0.56 gb/mo * $0.09/gb  = $0.05/mo
- 50gb storage * $0.023 per GB = $1.15/mo

Total S3 costs: $1.41/mo

## Other costs

GoDaddy DNS registration: $18/year (reserving redditloans.com)

DNS Privacy registration: $10/year (reduces how much email spam I get somewhat)

$28/year = $2.33/mo

## Total Cost Estimates

- EC2 Instances: $60.74/mo
- EC2 Data Transfer Out: $7.25/mo
- S3: $1.41/mo
- Website Registration: $2.33/mo

Estimated net before taxes: $71.73/mo

Estimated taxes (11%): $7.89/mo

Estiamted net after taxes: $79.62/mo
