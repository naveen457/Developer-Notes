# Domains and DNS: System Design Notes

## Running example: Namecheap + `astrix-app.me`

> **How to use these notes:** Each section starts with the general
> concept, then connects it to a Namecheap-style configuration and the
> `astrix-app.me` example. Exact record values depend on the DNS and
> hosting providers actually configured for the domain. Never copy
> illustrative IP addresses or placeholder email values into production.

------------------------------------------------------------------------

## 1. Domain registration vs. DNS hosting vs. web hosting

These are three different jobs that are often provided by the same
company, which can make them easy to confuse.

### Domain registration

A **registrar** is the company through which you register and renew a
domain name. It manages registration details and submits changes to the
registry for the relevant top-level domain.

Example: Namecheap may be the registrar for `astrix-app.me`. The
domain's registration gives you the right to use that name while it
remains registered and compliant with the applicable terms.

Registration does **not** automatically mean Namecheap hosts your
website or is the DNS provider currently answering queries.

### DNS hosting

A **DNS hosting provider** stores and serves the authoritative DNS
records for a domain. Those records tell resolvers where to find
services such as a website, API, or email system.

Namecheap can provide DNS hosting, but the domain could instead use
nameservers operated by a hosting platform or another DNS provider. The
configured nameservers determine which provider is authoritative.

### Web/application hosting

A **hosting provider** runs the frontend, backend, or other application
services. DNS points users toward the relevant endpoint; it does not run
the application.

For example, the frontend and backend might be deployed separately:

-   `astrix-app.me` → frontend hosting endpoint
-   `www.astrix-app.me` → frontend hostname or alias
-   `api.astrix-app.me` → backend hosting endpoint

The actual DNS record type and value must come from each hosting
provider's custom-domain instructions.

### How the pieces connect

1.  A user enters `astrix-app.me` in a browser.
2.  DNS resolution finds the relevant IP address or hostname.
3.  The browser connects to the destination using HTTP or HTTPS.
4.  TLS verifies the website's certificate.
5.  The hosting platform, reverse proxy, or web server routes the
    request to the application.

**System Design takeaway:** Registrar, authoritative DNS, and
application hosting are separate roles. A problem in any one can make a
site unavailable, but the symptoms and fixes differ.

------------------------------------------------------------------------

## 2. Nameserver delegation and glue records

### What is a nameserver?

A nameserver answers DNS questions. An **authoritative nameserver**
publishes the official records for a DNS zone. A **recursive resolver**
looks up answers for clients and caches results.

A domain's delegation tells the DNS hierarchy which nameservers are
authoritative for it.

### How delegation works

For a name such as `astrix-app.me`, a resolver typically follows this
chain:

1.  It asks a root server where to find the `.me` top-level-domain
    servers.
2.  It asks a `.me` server for the delegation for `astrix-app.me`.
3.  The `.me` server returns the delegated nameservers (NS records, plus
    glue where needed).
4.  The resolver queries one of the domain's authoritative nameservers
    for the requested record, such as an A record.

Root servers do **not** normally store the website's A record. They
direct the resolver to the next level.

### Namecheap: BasicDNS vs. Custom DNS

In Namecheap, the nameserver selection is commonly found under **Domain
List → Manage → Nameservers**. The precise labels can change over time.

-   **Namecheap BasicDNS or another Namecheap-operated DNS option:**
    manage records in the corresponding Namecheap DNS interface,
    typically Advanced DNS → Host Records.
-   **Custom DNS / third-party nameservers:** the selected external DNS
    provider is authoritative. Add or edit records there, not in a
    Namecheap Host Records panel that is no longer authoritative.

Before troubleshooting records, first confirm which nameservers the
domain actually delegates to. Editing records in the wrong dashboard
will not change the public authoritative answer.

### NS records

NS records identify the nameservers responsible for a zone. They are
different from A records: an A record maps a hostname to an IPv4
address, while an NS record identifies DNS servers for a zone.

### Glue records

A glue record is address information published by the parent zone to
help resolve a nameserver hostname when that hostname is inside the
domain it serves.

Example of the circular dependency:

-   The domain is `example.com`.
-   Its nameserver is `ns1.example.com`.
-   To contact `ns1.example.com`, a resolver needs its IP.
-   But finding that IP through `example.com` would require contacting
    the nameserver in the first place.

The parent zone can publish **glue** for `ns1.example.com`, breaking the
loop. Glue is needed in certain in-bailiwick configurations; it is not
required for every custom nameserver arrangement.

### Failure implications

If delegation points to incorrect or unreachable nameservers, the domain
may fail to resolve even if the correct A record exists in a dashboard
somewhere. A broken DNSSEC chain can also make a domain fail for
validating resolvers.

**System Design takeaway:** Delegation is the pointer to the DNS
authority. The record data matters only if it is published by the
authoritative provider the public DNS hierarchy actually reaches.

------------------------------------------------------------------------

## 3. DNS failure scenarios and troubleshooting

Different DNS errors indicate different failure modes. Diagnosing the
exact error is more useful than repeatedly changing records at random.

### Common symptoms

  -----------------------------------------------------------------------
  Symptom                 Meaning in broad terms  Common causes
  ----------------------- ----------------------- -----------------------
  `NXDOMAIN`              The queried name does   Typo, missing
                          not exist according to  record/name, wrong
                          the response            zone, negative cache

  `SERVFAIL`              Resolver could not      DNSSEC validation
                          complete resolution     problem, authoritative
                          successfully            failure, broken
                                                  delegation, upstream
                                                  issue

  Timeout                 No response arrived     Network filtering,
                          within the expected     unreachable server,
                          time                    packet loss, server
                                                  outage

  Wrong IP returned       DNS answered, but with  Incorrect record, stale
                          an unexpected address   cache, wrong
                                                  authoritative provider

  Root domain works,      Different hostnames     Missing or incorrect
  `www` fails             have separate DNS       `www` record
                          configuration           

  Website resolves but    DNS may be working;     Hosting outage,
  won't load              later connection layers firewall, TLS, reverse
                          fail                    proxy, app error

  Website works but API   Frontend DNS may be     API DNS, CORS, TLS,
  fails                   fine                    backend health, route
                                                  configuration
  -----------------------------------------------------------------------

### Useful commands

#### Windows

``` powershell
nslookup astrix-app.me
nslookup www.astrix-app.me
nslookup api.astrix-app.me
ipconfig /displaydns
ipconfig /flushdns
```

`ipconfig /flushdns` clears the local Windows DNS client cache. It does
**not** clear caches held by public recursive resolvers, browsers, other
devices, or authoritative DNS servers.

#### macOS/Linux with `dig`

``` bash
dig astrix-app.me A
dig astrix-app.me AAAA
dig www.astrix-app.me CNAME
dig astrix-app.me NS
dig astrix-app.me MX
dig +trace astrix-app.me
```

To query a specific recursive resolver:

``` bash
dig @1.1.1.1 astrix-app.me A
dig @8.8.8.8 astrix-app.me A
```

To query a known authoritative nameserver directly:

``` bash
dig @ns1.example-dns-provider.com astrix-app.me A
```

Replace the sample nameserver with one returned by the domain's actual
delegation.

### A sensible troubleshooting sequence

1.  **Confirm the domain is registered and not expired.**
2.  **Check the delegated nameservers.** Verify that they match the DNS
    provider you intend to use.
3.  **Query the authoritative nameserver directly.** This helps
    distinguish incorrect published data from recursive cache effects.
4.  **Compare multiple recursive resolvers.** Differences can be caused
    by cache age or resolver behavior.
5.  **Inspect the record name, type, target, and TTL.**
6.  **Check DNSSEC** if authoritative answers appear correct but
    validating resolvers return `SERVFAIL`.
7.  **Test the destination itself.** If DNS returns the expected IP,
    investigate network connectivity, TLS, hosting, proxy, and
    application logs.
8.  **Avoid changing multiple unrelated records at once.** Record the
    old values so you can roll back.

**System Design takeaway:** DNS is one layer of a request path. A
successful DNS lookup proves only that the name was resolved; it does
not prove the website or API is healthy.

------------------------------------------------------------------------

## 4. HTTPS, TLS certificates, and CAA records

### DNS and HTTPS solve different problems

DNS answers: **Where should this hostname resolve?**

TLS answers: **Can the client establish an encrypted connection and
verify the server's identity for this hostname?**

A domain can resolve to the correct IP and still show a browser
certificate warning. Common reasons include:

-   The certificate is expired.
-   The certificate does not include the requested hostname.
-   The server is presenting the wrong certificate.
-   The certificate chain is incomplete or invalid.
-   The hosting provider has not finished validating the custom domain.
-   The DNS record points to an old host that serves a different
    certificate.

### What happens during HTTPS

1.  DNS resolution returns an address or hostname path.
2.  The browser connects to the server on the HTTPS endpoint, usually
    TCP port 443, or uses an applicable modern transport such as QUIC
    for HTTP/3.
3.  The TLS handshake negotiates cryptographic settings.
4.  The server presents a certificate.
5.  The browser verifies the certificate chain, validity, and hostname.
6.  After successful negotiation, application HTTP traffic is encrypted.

DNS itself does not issue or validate the website certificate.

### CAA records

A **CAA (Certification Authority Authorization)** record lets a domain
owner specify which certificate authorities are permitted to issue
certificates for the domain. It is a restriction mechanism, not a
certificate itself.

Illustrative record:

``` dns
example.com. CAA 0 issue "letsencrypt.org"
```

Only publish a CAA policy after checking which certificate authorities
your hosting platform uses. An overly restrictive or incorrect CAA
record can prevent certificate issuance or renewal.

### Namecheap and Astrix workflow

When adding `astrix-app.me` or a subdomain to a hosting provider:

1.  Add the custom domain in the hosting dashboard.
2.  Follow the provider's required DNS instructions.
3.  Create the specified A, AAAA, CNAME, or verification records in the
    authoritative DNS provider.
4.  Wait for the provider to validate the domain.
5.  Confirm the certificate is issued and HTTPS works.
6.  Test both the root domain and any `www` or API hostname separately.

Do not assume that pointing a domain to an IP automatically configures
HTTPS.

------------------------------------------------------------------------

## 5. DNS vs. HTTP redirects

A DNS record does not generally redirect a browser to a URL path. DNS
maps names to DNS data; HTTP redirects are responses sent by a web
server or forwarding service.

### DNS records vs. redirects

  ----------------------------------------------------------------------------------------------
  Mechanism               What it does            Example
  ----------------------- ----------------------- ----------------------------------------------
  A                       Maps a hostname to an   `app.example.com → 192.0.2.10`
                          IPv4 address            

  AAAA                    Maps a hostname to an   `app.example.com → 2001:db8::10`
                          IPv6 address            

  CNAME                   Makes one DNS name an   `www.example.com → example-host.example.net`
                          alias of another        
                          hostname                

  URL redirect / HTTP     Sends a client to       `example.com → https://www.example.com/`
  redirect                another URL             
  ----------------------------------------------------------------------------------------------

An ordinary CNAME cannot specify a URL path such as `/login`, a protocol
such as `https://`, or an HTTP status code.

### 301 vs. 302

-   **301 Moved Permanently:** indicates that the resource has
    permanently moved. Browsers and search engines may cache or remember
    the redirect.
-   **302 Found:** indicates a temporary redirect. Other redirect status
    codes, such as 307 and 308, have related but distinct
    method-preservation semantics.

Redirect behavior is provided by an HTTP server, CDN, or
domain-forwarding service, not by a standard A record.

### Namecheap URL Redirect Record

Namecheap's interface may offer a **URL Redirect Record**. This
configures a forwarding behavior through the provider's service; it is
not the same as pointing a hostname directly at an application server.

Be careful with masked forwarding. It may use frames or other mechanisms
that can complicate HTTPS, browser behavior, accessibility, and search
indexing.

### Example

If you want `www.astrix-app.me` to show the same site as
`astrix-app.me`, a CNAME to the root name may be appropriate if
supported by the DNS setup and hosting platform. If you want one URL to
send the browser to a different URL, use an HTTP redirect or an
appropriate forwarding feature.

------------------------------------------------------------------------

## 6. DNS propagation vs. nameserver changes

### What people mean by "propagation"

"DNS propagation" is a common phrase for the time it takes different
resolvers and clients to begin seeing updated DNS information. It is not
a single global push process in which every DNS server receives the
change simultaneously.

Several mechanisms affect when an update is observed:

-   The authoritative provider must publish the change.
-   Recursive resolvers may have cached the old record.
-   Local operating systems and applications may cache answers.
-   Negative responses (such as a name not existing) can also be cached.
-   Nameserver changes affect delegation and can interact with cached NS
    and related records.
-   Provider-specific publishing and validation steps may add time.

### TTL example

Suppose `astrix-app.me` has an A record with TTL 86,400 seconds (24
hours). A recursive resolver caches the old answer just before the
record is changed. It may reuse that answer for close to the remaining
24-hour cache lifetime.

If the record's TTL is 300 seconds, a resolver that cached it under that
TTL will normally stop using the cached answer after five minutes. But
lowering the TTL **now** does not retroactively reduce the lifetime of
answers already cached with a 24-hour TTL.

### Safer migration plan

1.  Before the planned change, lower the TTL on the existing record to
    something like 300 seconds if your DNS provider permits it.
2.  Wait at least the previous TTL period so older long-lived cache
    entries have time to expire.
3.  Prepare the new server and verify it works before switching traffic.
4.  Update the authoritative record.
5.  Query the authoritative nameserver and several recursive resolvers.
6.  Keep the old server available during the transition when feasible.
7.  Restore a longer TTL after the migration is stable.

This reduces the expected transition time but does not guarantee every
client will switch immediately. Some resolvers may serve stale answers,
and some applications maintain their own connection or DNS caches.

### Why nameserver changes may take longer

Changing nameservers is different from editing an A record within the
same DNS provider. The parent zone must reflect the new delegation, and
resolvers may have cached the previous delegation or related data. The
new provider must also be correctly configured before the switch.

A practical migration is to copy and verify all required records at the
new provider **before** changing nameservers. Otherwise, the domain can
become unavailable even if the delegation itself updates successfully.

### Important distinction

-   **DNS update:** changes where a hostname resolves.
-   **Deployment:** changes application code or infrastructure.
-   **DNS caching:** may cause clients to keep using an older answer.
-   **HTTP/TLS caching and persistent connections:** can add separate
    behavior that DNS TTL does not control.

For quick server switching, a stable reverse proxy, CDN, or load
balancer is often better than repeatedly changing DNS records.

------------------------------------------------------------------------

## 7. CDN, reverse proxy, load balancing, and DNS failover

These technologies can work together, but they are not interchangeable.

### CDN

A **Content Delivery Network** serves content through a distributed
network of edge locations. It can reduce latency, cache static content,
terminate TLS, and provide security features. A CDN can also proxy
requests to an origin server.

### Reverse proxy

A **reverse proxy** accepts client requests and forwards them to backend
servers. Clients connect to the proxy rather than directly to each
backend.

Typical responsibilities include routing by hostname or path, TLS
termination, access control, and sometimes caching.

### Load balancer

A **load balancer** distributes requests across multiple backend
servers. Depending on the product and configuration, it can perform
health checks and stop sending new requests to unhealthy targets.

### DNS failover

DNS failover changes DNS answers based on a policy or health check. It
can direct new lookups toward a healthy endpoint, but cached answers
mean some clients may continue using the old endpoint for a period.

### Stable-endpoint architecture

``` text
Users
  |
  v
Stable public hostname
  |
  v
CDN / reverse proxy / load balancer
  |
  +------> Backend A (healthy)
  |
  +------> Backend B (healthy)
  |
  +------> Backend C (unhealthy; remove from rotation)
```

If the public hostname continues resolving to the same proxy or
load-balancer address, the provider can change backend routing without
requiring every user to learn a new DNS IP.

This is one reason production systems often use load balancers or
managed proxy platforms rather than manually changing A records for each
deployment.

### Failover is not magic

A load balancer must have a healthy alternate target. If all backends
fail, it cannot manufacture capacity. Long-lived connections may also
break when a backend is removed. Health-check intervals and thresholds
influence how quickly a failure is detected.

### Astrix connection

If the frontend and backend use separate hosting services, DNS maps each
hostname to the relevant service endpoint. A CDN or proxy may sit in
front of either service, depending on the deployment architecture. CORS,
authentication, and API route handling remain application/web concerns;
DNS alone does not configure them.

------------------------------------------------------------------------

## 8. Email DNS troubleshooting

Email depends on several DNS record types and on correct configuration
inside the email provider.

### MX: where incoming email goes

MX records specify mail exchangers for a domain. The preference value
indicates priority: a lower number is preferred over a higher number
when the relevant servers are available.

Illustrative example:

``` dns
example.com. MX 10 mail.provider.example.
```

The mail hostname must itself resolve appropriately. An MX record does
not create a mailbox; the receiving mail service must be configured to
accept the domain and recipient.

### SPF: authorized sending sources

SPF is published as a TXT record. It describes which sending systems are
authorized to send mail using the domain in the SMTP envelope sender.

Illustrative syntax:

``` dns
example.com. TXT "v=spf1 include:provider.example -all"
```

The `include` value must come from the actual mail provider. Do not copy
this example blindly.

**Important:** A domain should generally publish one SPF policy record
for a given name. Multiple separate SPF records can cause SPF evaluation
errors. Combine the mechanisms according to the providers' instructions.

### DKIM: message signing

DKIM adds a cryptographic signature to outgoing email. The sending
system keeps a private key and signs messages; the receiver uses the
public key published in DNS to verify the signature.

A DKIM record often lives under a selector name such as:

``` text
selector._domainkey.example.com
```

The selector and key are supplied by the email provider. A missing,
malformed, or outdated key can cause DKIM validation to fail.

### DMARC: policy and alignment

DMARC tells receivers how to handle mail that fails DMARC evaluation and
can enable aggregate reports. DMARC alignment relates the authenticated
SPF or DKIM domain to the visible From domain.

Illustrative record:

``` dns
_dmarc.example.com. TXT "v=DMARC1; p=none; rua=mailto:dmarc@example.com"
```

This is a monitoring-style example. Choose a policy deliberately, and
make sure the reporting address and DNS syntax are correct before
deployment.

### PTR / reverse DNS

A PTR record maps an IP address back to a hostname. Reverse DNS is
generally controlled by the owner of the IP address or its network
provider, not simply by the domain's regular Host Records panel.

PTR configuration can matter for mail-server reputation and validation,
especially when operating your own mail server. Managed email providers
typically handle the infrastructure-side reverse DNS for their sending
IPs.

### Troubleshooting email

1.  Confirm the domain's authoritative nameservers.
2.  Query MX records.
3.  Confirm that the mail provider has verified the domain and
    configured the mailbox.
4.  Check SPF and ensure there is only one SPF policy at the relevant
    name.
5.  Check the exact DKIM selector and public key supplied by the
    provider.
6.  Check DMARC syntax and alignment.
7.  If self-hosting mail, check PTR/reverse DNS and the provider's
    network requirements.
8.  Inspect mail-provider logs and spam/rejection messages; correct DNS
    alone does not guarantee delivery.

**Important distinction:** MX primarily controls incoming routing. SPF,
DKIM, and DMARC support authentication and anti-spoofing. Website DNS
can work even while email is broken, because the website and mail
records are independent.

------------------------------------------------------------------------

## 9. DNS monitoring, redundancy, and disaster recovery

DNS is part of the availability path. If authoritative DNS becomes
unreachable, resolvers may temporarily continue answering from cache,
but once cached data expires they may be unable to obtain fresh answers.

### Multiple nameservers

A zone is normally delegated to multiple authoritative nameservers. This
provides alternatives if one server or network path is unavailable.

However, listing two nameservers does not automatically guarantee
resilience if both depend on the same failed network, region, account,
or underlying provider. Redundancy is strongest when failure domains are
genuinely independent.

### DNS-provider redundancy

Some organizations use more than one authoritative DNS provider. This
can reduce dependency on a single provider but requires careful
synchronization of records, DNSSEC configuration, and change processes.
A poorly synchronized secondary provider can publish stale or
inconsistent data.

### Backups and recovery

Maintain an export or documented copy of the DNS zone, including:

-   A and AAAA records
-   CNAME or ALIAS records
-   MX records
-   SPF, DKIM, and DMARC TXT records
-   Domain verification TXT records
-   CAA and other policy records
-   Relevant subdomain and service records

Also record which provider is authoritative, how the domain's
nameservers are configured, and where registrar account recovery
information is kept securely.

A DNS zone backup is not a substitute for registrar-account security.
Protect the registrar with strong authentication, unique credentials,
and appropriate account recovery controls.

### Monitoring

Monitor more than one thing:

-   Does the domain resolve through multiple recursive resolvers?
-   Does the authoritative DNS provider respond?
-   Does HTTPS present a valid certificate?
-   Does the frontend return a healthy response?
-   Does the backend/API respond correctly?
-   Are email records still present if email is important?

A DNS check can pass while the application is down, so monitor
application health separately.

### Disaster-recovery plan

1.  Keep a record of known-good DNS values.
2.  Ensure an alternate application endpoint is ready if high
    availability is required.
3.  Verify the current nameserver delegation.
4.  Restore missing or incorrect records at the authoritative provider.
5.  If switching DNS providers, preconfigure and test the destination
    zone before changing delegation.
6.  Check DNSSEC DS and signing configuration if DNSSEC is enabled.
7.  Verify from multiple resolvers and test the actual application.
8.  Keep the old endpoint active during cache transitions when possible.

------------------------------------------------------------------------

## 10. Namecheap-specific practical checklist for `astrix-app.me`

The following is a diagnostic checklist, not a claim about the domain's
current configuration. The actual setup must be verified in the
Namecheap account and the hosting dashboards.

### A. Confirm the authoritative DNS provider

1.  Sign in to Namecheap.
2.  Open **Domain List → Manage** for `astrix-app.me`.
3.  Inspect the **Nameservers** setting.
4.  Identify whether Namecheap DNS or a third-party provider is
    authoritative.
5.  Make record changes in the authoritative provider's dashboard.

### B. Review Host Records

In the Namecheap interface, the usual path for supported Namecheap DNS
configurations is:

**Domain List → Manage → Advanced DNS → Host Records**

Review each record's type, host, value, and TTL. Common record types
include A, AAAA, CNAME, MX, TXT, SRV, CAA, and URL Redirect, subject to
provider support and current UI behavior.

### C. Map hostnames to services

Write down the intended mapping before changing anything:

  -----------------------------------------------------------------------
  Hostname                Intended service        Required DNS value
  ----------------------- ----------------------- -----------------------
  `astrix-app.me`         Frontend                Exact target specified
                                                  by frontend host

  `www.astrix-app.me`     Frontend alias or       Exact target or
                          redirect                redirect configuration

  `api.astrix-app.me`     Backend                 Exact target specified
                                                  by backend host

  Mail-related names      Email provider          Exact MX/TXT values
                                                  from mail provider
  -----------------------------------------------------------------------

Do not guess hosting IPs or CNAME targets. Some providers use changing
IP ranges, and some require a hostname rather than a fixed IP.

### D. Validate a change

Before editing, save the existing values. After editing:

``` bash
dig astrix-app.me A
dig astrix-app.me AAAA
dig www.astrix-app.me A
dig www.astrix-app.me CNAME
dig astrix-app.me NS
dig astrix-app.me MX
```

Not every query will return a record: for example, a CNAME query may be
empty when `www` uses an A record or a provider-specific ALIAS
arrangement. Interpret results according to the actual configuration.

Then test:

-   The frontend URL over HTTPS
-   The API hostname over HTTPS
-   The application's expected API routes
-   Browser console errors, including CORS or TLS errors
-   Hosting logs and health checks

If DNS resolves to the intended target but the API request fails, do not
keep changing DNS automatically. Check the application URL, CORS policy,
TLS certificate, routing, and backend availability.

### E. Before changing nameservers

1.  Export or record every existing DNS record.
2.  Recreate all required records at the destination provider.
3.  Confirm website, API, email, and verification records are present.
4.  Check DNSSEC status and coordinate DS/signing changes if applicable.
5.  Change delegation only after the destination zone is ready.
6.  Verify the result from authoritative and recursive queries.

------------------------------------------------------------------------

## 11. Additional foundations worth remembering

### The 13 logical DNS root server identities

There are 13 named root-server identities, conventionally labeled A
through M. They are **not merely 13 physical machines**: each identity
is served by many distributed instances using anycast and other
infrastructure. Root servers provide referrals to top-level-domain
servers; they do not maintain every website's A record.

### DNS caching and the meaning of TTL

TTL is the cache lifetime associated with a DNS record. A resolver
decrements the remaining TTL as time passes. Once the cached answer
expires, the resolver ordinarily needs to refresh it before reusing it.
Resolver policies, serve-stale behavior, local caches, and application
behavior can affect what a user observes.

### Dynamic DNS

Dynamic DNS automatically updates a DNS record when a changing IP
address changes. It is useful for home servers and dynamic ISP
addresses, but it does not defeat caching and cannot bypass
carrier-grade NAT by itself.

### Personal DNS servers

-   **Authoritative server:** publishes official records for a zone.
-   **Recursive resolver:** resolves names for clients and caches
    answers.
-   **Private/internal DNS:** resolves internal names for a network,
    often over a VPN or within a home lab.

These roles have different security and availability requirements. Avoid
exposing an open recursive resolver to the public internet.

### DNSSEC

DNSSEC adds cryptographic authenticity and integrity checks to DNS data
through signatures and a chain of trust. It does not encrypt DNS
queries. Incorrect DS or signing configuration can make validating
resolvers reject answers.

------------------------------------------------------------------------

## 12. Interview and System Design questions

1.  What is the difference between a registrar, DNS host, and
    application host?
2.  What happens after a user enters a domain name into a browser?
3.  What is the difference between an authoritative server and a
    recursive resolver?
4.  What are the 13 root-server identities, and why are they not just 13
    physical servers?
5.  What is TTL, and why does lowering TTL immediately before a
    migration not instantly invalidate old cache entries?
6.  Why can a DNS change take hours even after the dashboard reports
    success?
7.  Why is changing nameservers riskier than changing one A record?
8.  How would you migrate a domain to a new IP with minimal downtime?
9.  Why can a site resolve correctly but still fail HTTPS?
10. What are the roles of MX, SPF, DKIM, and DMARC?
11. What does DNSSEC protect against, and what can go wrong with a bad
    DS record?
12. What is the difference between a CNAME and an HTTP redirect?
13. Why might DNS failover be slower or less reliable than a load
    balancer?
14. How would you investigate `NXDOMAIN`, `SERVFAIL`, and a timeout?
15. How would you design DNS redundancy and a recovery plan for a
    critical application?

------------------------------------------------------------------------

## Final mental model

Think of the request path as a series of separate systems:

**Domain registration → nameserver delegation → authoritative DNS
records → recursive resolver/cache → IP/hostname destination → network
connection → TLS/HTTPS → proxy/load balancer → frontend/backend
application.**

When something fails, identify the layer first. DNS controls name
resolution; it does not guarantee that the destination is reachable,
that the TLS certificate is valid, that the API is healthy, or that an
email will reach the inbox.
