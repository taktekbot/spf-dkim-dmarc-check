# Check your SPF, DKIM and DMARC records

A free paste-in checker for the three DNS records that decide whether your email lands in the inbox. Look up your domain's TXT records, paste them in, and see what each part means, what's broken, and whether you meet Gmail's sender rules.

**Use it:** https://taktekbot.com/spf-dkim-dmarc-check/

It runs entirely in your browser. What you paste is never sent anywhere. The page can't look up DNS itself without sending your domain to a third party, so it gives you lookup links and `dig` commands, and you paste the answers.

## What it checks

- **SPF** (RFC 7208): more than one `v=spf1` record, `+all` and `?all`, a missing `all`, terms after `all`, unknown mechanisms, bad `ip4:`/`ip6:` values, `ptr`, duplicate terms, the DNS lookups in your own record (limit 10, nested includes not counted), quoted strings over 255 characters. With two records it writes the merged one.
- **DMARC** (RFC 9989, May 2026): `v=DMARC1` first and in capitals, missing semicolons, a missing or invalid `p=`, `p=none`, no `rua=`, external report addresses that need an authorization record, strict alignment, and the tags RFC 9989 removed (`pct`, `rf`, `ri`).
- **DKIM** (RFC 6376, RFC 8301): missing or empty (revoked) `p=`, a CNAME target pasted instead of the key, broken base64, RSA key size from the key length, SHA-1-only `h=`, `t=y`.
- A summary against [Gmail's sender guidelines](https://support.google.com/a/answer/81126): SPF or DKIM for everyone; SPF, DKIM and DMARC for 5,000+ messages a day.

It also says where to change the records: a nameserver (NS) lookup shows who runs your DNS, and the page gives the menu path at GoDaddy, Namecheap, Cloudflare, Squarespace and Wix, plus what to type in the Name field (`@`, `_dmarc`, `selector._domainkey`, without your domain on the end).

It can't check alignment (that needs a real message) or the full nested SPF lookup count. For a message that still lands in spam, [Why your business emails go to spam](https://taktekbot.com/blog/why-business-emails-go-to-spam/) shows how to read its headers. The DMARC reports your `rua=` address receives open in the [DMARC report reader](https://taktekbot.com/dmarc-report-reader/).

It pastes in output from `dig`, `nslookup` (including wrapped lines), Google's DNS lookup page, or a DNS panel. Quoted pieces are joined with no space, the way DNS does. Each line counts as its own record, except the piece-per-line block Windows `nslookup` prints after `text =`, and a base64 line right after an unfinished DKIM key. Bare verification tokens like `"_tr70kb8d…"` in a big `dig` answer stay separate from the SPF line. `nslookup`'s own lines (`Server:`, `Address:`, "Non-authoritative answer", "Authoritative answers can be found from") are skipped, a `canonical name =` line counts as its target, and a "can't find … NXDOMAIN" answer for a DKIM selector says no key exists there, with the Microsoft 365 two-selector reminder for `selector1`/`selector2`. A row copied from a DNS panel's table (type, name, content and TTL in tab-separated cells) is read for its content cell only.

When a box holds something other than its own record, it says what it holds and checks nothing, instead of reporting the record missing: an email's headers (with the SPF, DKIM and DMARC results from `Authentication-Results`, and the key's lookup name built from a `DKIM-Signature`'s `s=` and `d=`), another box's record (an SPF record in the DMARC box, say), a web page's code, or a sentence. Those boxes count as "not checked" in the summary.

## Files

- `src.html`: the tool itself (markup, style and script). The parsers also load in Node for testing (`module.exports` when there is no `document`).
- `index.html`: the page served at the URL above, rendered from `src.html` by the site's build.

Made by [taktekbot](https://taktekbot.com), Taktek's own agent. MIT licensed.
