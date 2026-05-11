---
title: "Domain Name System (DNS)"
tags: [ "networking", "dns", "caching" ]
prerequisites:
  - "concepts/networking/network-layers.md"
status: "draft"
last_updated: "2025-10-05"
todos:
  - "Deep dive into DNS record types (A, AAAA, CNAME, MX, NS, etc.)."
  - "Add a diagram illustrating the DNS resolution process."
  - "Explain DNS propagation."
---

## Summary

The Domain Name System (DNS) is a hierarchical and decentralized naming system that acts as the Internet's phonebook. It translates human-readable domain names (e.g., `www.example.com`) into machine-readable IP addresses (e.g., `93.184.216.34`), allowing users to access websites and services without memorizing complex numerical addresses. This process, known as DNS resolution, is a fundamental component of how the internet works.

## The DNS Resolution Process

The process involves a chain of servers, each responsible for a different part of the domain name.

1.  **Resolver:** The process begins with a DNS resolver (or recursive resolver), typically operated by an Internet Service Provider (ISP) or a third-party provider like Google (8.8.8.8) or Cloudflare (1.1.1.1). When you request a domain, the resolver's first step is to check its own cache to see if it already has the IP address. If not, it begins a recursive query.
2.  **Root Nameservers:** The resolver queries one of the 13 sets of root nameservers. These servers don't have the IP address, but they know who is responsible for the next level, the Top-Level Domain (TLD), and direct the resolver there.
3.  **TLD Nameservers:** The TLD nameserver manages a specific top-level domain, like `.com`, `.org`, or `.net`. The resolver asks the `.com` TLD server for `example.com`. The TLD server responds with the location of the final authority: the domain's authoritative name server.
4.  **Authoritative Name Server:** This is the server that holds the official DNS records for the specific domain (`example.com`). It receives the query from the resolver and returns the correct IP address.

## DNS Caching

DNS caching is crucial for performance and efficiency. To prevent overwhelming the root and authoritative nameservers, DNS responses are cached at multiple layers. When a user requests a domain, the query checks each layer of the cache before moving to the next, significantly speeding up response times and reducing network traffic.

### Layers of Caching

1.  **Browser Cache:** Modern web browsers are the first place a DNS query is checked. The browser maintains its own cache of recently visited domain names for a short period (determined by the record's TTL).
2.  **Operating System (Stub Resolver) Cache:** If the record is not in the browser's cache, the query goes to the operating system's DNS client, known as a "stub resolver." The OS maintains its own cache of DNS records.
3.  **Resolver Cache:** If the OS cache is empty, the query is sent to the configured DNS resolver (e.g., the ISP's server). This resolver has a larger, shared cache that serves many users, so it's more likely to have a recent, non-expired record for a popular domain.

If all cache layers are missed, the resolver then performs the full recursive lookup process described above.

### Time to Live (TTL)

TTL defines how long DNS records remain in cache before requiring a fresh lookup:

- **Short TTL** (5 minutes): Fast propagation of DNS changes, higher server load
- **Long TTL** (24 hours): Better performance and lower load, slower updates



## References

-   [Cloudflare: What is DNS?](https://www.cloudflare.com/learning/dns/what-is-dns/)
-   [AWS: What is DNS?](https://aws.amazon.com/route53/what-is-dns/)
