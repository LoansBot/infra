# SSH Bastion

When everything is operating normally, it's not possible to connect to any
server via SSH. Although it's extremely unlikely that someone would be able to
gain unauthorized access via SSH, doing so grants administrative privileges
and is hence an extremely high-value target. Thus it's absolutely essential
that any open SSH servers are running the latest security patches. To mitigate
risk, the redditloans VPC only ever allows SSH connections through a Bastion
server on the public subnet. All the other servers will accept SSH connections
from the Bastion server.

The Bastion server is terminated or shutdown when not actively being used.

## Instance Details

By default I choose t3a.nano with unlimited bursting disabled. Typically SSH is
not resource intensive, but it's nice if it has a reasonably strong connection
which correlates reasonably with low latency.

If it ever feels slow enabling unlimited bursting is very easy and applies
quickly.

I attach the minimum of 8gb of EBS space. This costs $0.80/mo, so it's nice
to terminate the instance if it's not expected to be needed for a while. It's
very fast to spin up new instances so I often just set the shutdown behavior
to terminate.

## Setup

Instances spawn with everything we need so it's just a matter of ensuring the
system has the latest updates not included in the AMI.

```bash
sudo yum -y update
```
