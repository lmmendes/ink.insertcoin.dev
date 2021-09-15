---
layout: post
pageId: 4
title:  home.arpa the TLD for home usage
comments: true
categories:
    - homelab
    - networking
    - tld
tags:
    - homelab
    - networking
    - tld
author: Luis Mendes
---

> **TLDR**: `.home.arp` is the special-use TLD that you should be using inside your private network according to [RFC 8375][rfc8375].

---

With my latest homelab rebuild I tried to make my network address more sane, and with that I needed to replace the `.local` TLD that I have been using for the last couple of years with one that had none of the gotchas that I sometimes found myself getting into.

`.local` is defined as a special-use TLD (see [RFC 6761][rfc6761]) in the [RFC 2606][rfc2606] suitable to be used for internal network usage but states that was designated to be used with Multicast DNS (mDNS), this means that each device in the network can grab and define its prefered hostname, making me loose control regarding the names that are registered in the network, and in the other side of things, while using `.local` TLD some DNS resolvers would defer the resolution the their mDNS resolver instead of the DNS causing resolution conflits.

 [RFC 8375][rfc8375] solves this by defining `home.arpa` has the de facto TLD for home usage:

 > This document registers the domain 'home.arpa.' as a special-use
   domain name [RFC 6761][rfc6761] and specifies the behavior that is expected
   from the Domain Name System with regard to DNS queries for names
   whose rightmost non-terminal labels are 'home.arpa.'.  Queries for
   names ending with '.home.arpa.' are of local significance within the
   scope of a homenet, meaning that identical queries will result in
   different results from one homenet to another.  In other words, a
   name ending in '.home.arpa.' is not globally unique.

Long story short, DNS resolutions for `*.home.arpa` hosts will not be send to public Internet, hence not leeking any kind of information about your internal network outside, and using `.home.arpa` will also ensure you don't have issues with possible mDNS resolution.

**Sidenote**:

- I would suggest not using `.home`, `homenet`, `workgroup` or other "fake" domain name that comes to mind because of two possible issues:
  - a) If the DNS query can't be solved inside your network, the request will be forwared to a root DNS server leeking information of your internal network outside, while also adding a delay penalty for the resolution.
  - b) [ICANN decided to delegate new TLDs][newtlds] to anyone who was willing to apply and pay the required fees,  this means that in the future anyone can register your internal domain and start using it has a root TLD, see what happend with the `.dev` domain that got bought by Google a couple years ago. 
- I would suggest avoiding using a public registered domain or subdomain inside your home network, just to keep the overhead low, because this would mean that you need to update the public DNS server each time you had a device to your network (eg: Cloudflare or DynDNS), and this would also be leeking information about your home network.

Sources:


- [RFC 6761][rfc6761]: Special-Use Domain Names
- [RFC 6762][rfc6762]: Multicast DNS
- [RFC 8375][rfc8375]: Special-Use Domain 'home.arpa.'

[rfc2606]: https://tools.ietf.org/html/rfc2606
[rfc6761]: https://tools.ietf.org/html/rfc6761
[rfc6762]: https://tools.ietf.org/html/rfc6762
[rfc8375]: https://tools.ietf.org/html/rfc8375
[newtlds]: https://newgtlds.icann.org/en/