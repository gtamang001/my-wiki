# Networking Commands for Platform Engineers

**Type:** Practical reference.

| Command | Use |
| --- | --- |
| `dig example.com` | Detailed DNS answer |
| `nslookup example.com` | Basic DNS resolution |
| `curl -v https://example.com` | HTTP, TLS and connection detail |
| `curl -I https://example.com` | HTTP response headers |
| `ss -lntp` | Listening TCP sockets |
| `ip route` | Host routing table |
| `ip addr` | Interface addresses |
| `traceroute example.com` | Path investigation where allowed |

## Layer-by-layer triage
1. **DNS:** Did the name resolve to the expected address?
2. **Transport:** Could the client connect to the destination port?
3. **TLS:** Did the handshake and certificate validation succeed?
4. **HTTP:** What response status and headers came back?
5. **Application:** Are backend endpoints ready and logs consistent with the error?

Some minimal containers omit these tools. Use an approved ephemeral debug container or disposable lab pod rather than modifying production images.
