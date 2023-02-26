# DNS 101

## What does DNS do?

- Translate domain names to IP addresses to return to requesters/clients

## Why DNS structure complex?

- Why not one server?
  - Single point of failure
  - Millions of domains. Each domain can contain hundreds, thousands of records
- Increase servers?
  - Scaling issues
  - Random request => each server need to contain all records
- Terms
  - Zone: Database. Exp: netflix.com
  - Zonefile: File storing the zone
  - NS server: a DNS server hosts zones => stores zonefiles
  - Authoritative: contains real records of a domain
  - Non-authoriative/cached: copies of records/zones stored elsewhere to speed things up
- Hierarchical Design
  - DNS root => DNS root servers
  - 13 root servers
  - ICANN operates one of 13 root server
  - Database managed by IANA
  - Root zone contains TLDs of DNS
    - generic: .com
    - country: .uk, .vn
  - IANA delegates management of TLDs to registries
    - .com => Verisign
  - Registries host name servers maintaining TLD zone
  - Root zone has records pointing .com NS to registries IP address server
  - .com TLD zone hosts records pointing to Authoritative NS servers maintaining dns records of domains, like netflix.com, apple.com
  - Authoritative NS servers hosts dns records of specified domains
    + Authoritative NS servers contain ZoneFile for the domain zone
