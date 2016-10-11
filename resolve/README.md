## Mass OWA Lookup

There are 3 ways to tell (remotely) if a domain is using Outlook Web Access:

1. GET https://domain.com/autodiscover/autodiscover.xml
2. GET https://autodiscover.domain.com/autodiscover/autodiscover.xml
3. Use DNS SRV record to get autodiscover address

Here we will use mass parallel DNS SRV lookups (using option 3 above). We modified some code from @majek to lookup SRV records.

Given a file with domains, run this command to prepend '_autodiscover._tcp.':

```sed -i.bak 's/^/_autodiscover._tcp./' domainlist.txt```

Then we perform the lookups:

```./resolve -srv < domainlist.txt > domainlistOWAResults.txt```

The non-SRV version of this script should be able to resolve a million domains in 25 minutes on an Amazon box [1]. With our SRV records it took 6 minutes to check 35,000 domains, which seems a little slow:

```
[ec2-user@ip-172-31-30-50 ~]$ wc -l domainlist.txt
35880 domainlist.txt
[ec2-user@ip-172-31-30-50 ~]$ sed -i.bak 's/^/_autodiscover._tcp./' domainlist.txt
[ec2-user@ip-172-31-30-50 ~]$ ./resolve -srv -server 172.31.0.2:53 < domainlist.txt > domainlistOWAResults.txt
Server: 172.31.0.2:53, sending delay: 8.333333ms (120 pps), retry delay: 1s
Resolved 35880 domains in 339.024s. Average retries 1.162. Domains per second: 105.833

[ec2-user@ip-172-31-30-50 ~]$ cat domainlistOWAResults.txt | cut -d "," -f 2 | sort | uniq | wc -l
460
```

So of the 35k domains, 460 are using OWA.

FYI, don't use Google (8.8.8.8 etc) as your lookups will be throttled - Amazon seems to be pretty zippy with the DNS provided on their boxes.

### References
[1] https://idea.popcount.org/2013-11-28-how-to-resolve-a-million-domains/
