# 1.0.1 - How a Request Travels

## The problem

You enter `shop.example` and the page takes six seconds to appear. Where did the time go? Trace one page load from the browser to the server and back, then look for the slow part.

## The request path

1. **DNS finds an address.** The browser asks a DNS resolver for the IP address associated with `shop.example`. DNS is the naming system; the answer is an IP address. The lookup might take 20 ms. DNS answers are cached for their TTL, so a repeat visit may skip a fresh lookup while the answer remains valid.

2. **The browser connects.** It establishes a connection to the server. A round trip (RTT) is the time for a message to travel to the other side and for a reply to return. For this example, assume a 30 ms RTT and two round trips for connection setup: about 60 ms. HTTPS setup details vary; this is an illustration, not a universal handshake time.

3. **The browser requests the page.** It sends `GET /products`. The server runs the route's code. The request and response also cross the network, and the server spends time processing the request.

4. **The server fetches data.** If product data lives in a database, the server queries it and waits for the result. Keeping application servers stateless while storing durable data separately makes it easier to run multiple server instances and scale them. A slow query can dominate the response time.

5. **The server responds.** It uses the data to build a response, often HTML, and sends it to the browser. The browser parses the HTML and renders the page.

6. **The browser fetches page assets.** HTML may reference images, stylesheets, scripts, and fonts, which trigger more requests. HTTP/1.1 can use multiple connections, though requests on a connection can be constrained by ordering. HTTP/2 multiplexes many request streams over one connection; HTTP/3 also supports multiplexing over QUIC. Neither makes network distance or server work disappear.

## Why the page can be slow

- **Slow database query:** a recommendation query that sorts every product on every page view might add 400 ms. The same network path can still feel much slower because the server is waiting on data.
- **Far-away server:** a user in Sydney talking to a server in Virginia has a much higher RTT than a nearby user. If three network round trips rise from 30 ms to 200 ms each, that part grows from about 90 ms to 600 ms. The exact page slowdown depends on all other work and which requests can overlap.

## A useful latency model

For an illustrative 155 ms load, about 25 ms might be server and database work, with roughly 130 ms spent on DNS, connection setup, network exchanges, and transfer. These are example numbers, not a universal breakdown. Measure the actual page to find its bottleneck.

**Intuition:** A request is a series of waits across the browser, network, application, and database. Application code may be a small part of total latency; each network round trip and slow dependency can add up. Reuse connections, cache where appropriate, and reduce unnecessary waits.

## Quick recap

- DNS maps a name to an IP address, and its answer can be cached.
- Connections and request-response exchanges cost round trips.
- HTTP/2 and HTTP/3 can multiplex requests, but cannot remove latency.
- Slow database work can dominate a page load.
- Greater distance raises RTT and can multiply the cost of sequential exchanges.
