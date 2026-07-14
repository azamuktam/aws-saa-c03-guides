# Section 8: CloudFront & Global Accelerator

## The idea

Here's the physics problem AWS can't wish away: **far away = slow**. Your app lives in us-east-1, your user sits in Singapore, and every request has to cross half the planet — hundreds of milliseconds of lag, packet loss on the choppy public internet, unhappy users.

AWS gives you **two opposite strategies** to fight distance, and the exam loves testing whether you can tell them apart:

1. **CloudFront: bring the CONTENT closer.** Cache copies of your files at **400+ edge locations** worldwide. Think of a **bookstore chain**: instead of every customer flying to one warehouse, the chain stocks the popular books on the shelves of local stores. First customer waits for delivery (cache miss); everyone after grabs it off the shelf (cache hit).
2. **Global Accelerator: make the JOURNEY faster.** Nothing is cached, nothing is copied. Instead, the user's traffic **enters the AWS private backbone at the nearest edge location** and rides AWS's own fiber the rest of the way — a **private highway** instead of the congested public internet. Same warehouse, much better road.

```
CloudFront:            Global Accelerator:
User → edge (CACHED!)  User → nearest edge → AWS backbone → your app
       done.                   (in the path, no caching, every packet)
```

Content closer vs journey faster. Hold on to that, and most questions solve themselves.

## CloudFront (CDN — Content Delivery Network)

- **Origins** = where CloudFront fetches the real content: **S3 buckets, ALBs, or any HTTP server** (even outside AWS).
- **TTL (Time To Live)** = how long an edge keeps a cached copy. **Longer TTL → fewer trips to the origin → less origin load** ("reduce load on origin" = increase TTL / improve caching).
- **Invalidation** = force-expire cached content early (e.g. `/*` for everything). It **costs money** — the cheap habit is versioned filenames (`app-v2.js`) instead.
- **CloudFront also accelerates DYNAMIC content** — even uncacheable requests benefit from riding AWS's network from the edge to the origin. Don't dismiss CloudFront just because the content isn't static.

**Locking the bucket: OAC.** If users can hit your S3 bucket directly, they bypass CloudFront (and your caching, and your paywall). **OAC (Origin Access Control)** makes the bucket private and grants CloudFront alone the right to read it.

**THE trap:** *"S3 content must be accessible ONLY via CloudFront"* → **OAC**. You'll see **OAI (Origin Access Identity)** as a distractor — it's the **legacy** mechanism; OAC is the modern answer.

**Paid/private content:**

| Mechanism | Scope | Signal |
|---|---|---|
| **Signed URL** | **ONE file** | "give a customer access to a single video/file" |
| **Signed Cookies** | **MANY files** | "subscribers can access the whole premium library" |

**Geo Restriction** = allow-list or block-list entire **countries** at the edge. Signal: *"licensing agreements"*, *"block users from country X"*.

**THE trap:** *"custom SSL certificate for a CloudFront distribution"* → the ACM certificate **must be in us-east-1**, no matter where anything else lives. Free exam point.

**Field-level encryption** = encrypt specific sensitive form fields (credit card number) **at the edge**, so only your final application — not even your origin infrastructure — can decrypt them.

## Global Accelerator

- Gives you **2 static anycast IP addresses** — the same two IPs work worldwide, and they **never change**. Anycast means the internet automatically routes each user to the *nearest* AWS edge announcing those IPs. **Static IPs = the whitelisting answer**: when a client's firewall team demands fixed IPs to allow, CloudFront (thousands of shifting edge IPs) can't help; GA can.
- Handles **ANY TCP or UDP traffic** — not just HTTP. Signal words: **gaming (UDP!), VoIP, IoT, MQTT**.
- **No caching, ever.** GA is a road, not a warehouse.
- **Instant failover (~30 seconds)**: GA sits **in the request path** and health-checks your endpoints, so when a region dies it reroutes immediately — **no DNS-cache lag**, because clients keep using the same two IPs the whole time. This is exactly the weakness of Route 53 failover (clients cache old DNS answers) solved.

## THE comparison table

| | **CloudFront** | **Global Accelerator** |
|---|---|---|
| Strategy | Content closer (cache) | Journey faster (backbone) |
| Protocol | HTTP/HTTPS | **Any TCP/UDP** |
| Caching | **Yes** — that's the point | **No** — never |
| Static IPs | No (many changing edge IPs) | **Yes — 2 static anycast IPs** |
| Failover | n/a (origin failover exists, but) | **In-path, ~30s, no DNS lag** |
| Use cases | Websites, video, APIs, downloads | Gaming, VoIP, IoT, whitelisting, instant multi-region failover |

## The three-way direction trap

The exam loves this triangle — read the **direction and protocol** of the traffic:

```
Global users DOWNLOAD/view content (HTTP)   → CloudFront
Global users UPLOAD to an S3 bucket         → S3 Transfer Acceleration
Global users use a TCP/UDP app (non-HTTP)   → Global Accelerator
```

(S3 Transfer Acceleration uses the same edge network, but for **uploads into S3** — "users worldwide upload large files to a central bucket" is its exact sentence.)

### Two easy-to-miss CloudFront features (practice-test lessons)

- **Origin group** = a **primary + secondary origin** pair: when the primary returns errors, CloudFront **automatically retries against the secondary**. THE answer to "configure CloudFront for high availability / origin failover." (Geo restriction next to it is the false twin — that's country blocking, not HA.)
- **Price class** = which **edge locations** your distribution uses (all = best latency, fewer = cheaper). It's a **cost dial** — it has nothing to do with routing to origins. "Reduce CloudFront costs, tolerate higher latency in some regions" → price class.

## Question patterns

> *"Serve a static website to users worldwide with low latency; the S3 bucket must not be publicly accessible"* → **CloudFront + S3 origin + OAC** (cache at edges, lock the bucket to CloudFront only).

> *"Paying subscribers should stream any video in the premium catalog"* → **Signed Cookies** (MANY files = cookies; ONE file = signed URL).

> *"Send a customer a link to download a single report"* → **Signed URL** (one file, one URL).

> *"Multiplayer game over UDP has high latency for global players"* → **Global Accelerator** (UDP rules out CloudFront instantly).

> *"Enterprise clients' firewalls require a small set of FIXED IP addresses to whitelist"* → **Global Accelerator** (2 static anycast IPs — the only global service that offers this).

> *"Multi-region app needs failover in seconds, without waiting for DNS caches"* → **Global Accelerator** (in-path health checks, same IPs before and after).

> *"Licensing forbids streaming the content in certain countries"* → **CloudFront Geo Restriction** (country-level block at the edge).

> *"Reduce the load on the origin server behind CloudFront"* → **Increase TTL / improve cache hit ratio** (more hits at the edge = fewer origin fetches).

> *"Users around the world upload large files into one S3 bucket, uploads are slow"* → **S3 Transfer Acceleration** (UPLOAD direction — not CloudFront).

> *"Attach a custom domain certificate to a CloudFront distribution"* → **ACM certificate in us-east-1** (always, regardless of your app's region).

## Pocket card

| Keyword in question | Answer |
|---|---|
| global static content, low latency | **CloudFront** |
| "S3 only via CloudFront", private bucket | **OAC** (OAI = legacy distractor) |
| one paid file | **Signed URL** |
| whole paid library | **Signed Cookies** |
| block/allow countries | **Geo Restriction** |
| CloudFront custom cert | **ACM in us-east-1** |
| encrypt one sensitive field end-to-end | Field-level encryption |
| reduce origin load | Longer TTL / better caching |
| force-refresh cached files | Invalidation (`/*`, costs money) |
| UDP / gaming / VoIP / non-HTTP | **Global Accelerator** |
| fixed IPs for whitelisting | **Global Accelerator** (2 anycast IPs) |
| instant failover, no DNS-cache lag | **Global Accelerator** |
| global UPLOADS to S3 | **S3 Transfer Acceleration** |

Both of these services exist to get users to your data faster — but the data itself usually lives in a database, and how you make *that* fast and failure-proof is Section 9's RDS and Aurora.
