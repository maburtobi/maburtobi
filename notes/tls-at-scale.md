# Distributing Let's Encrypt certificates to a fleet without breaking the keys

Every HTTPS host needs a valid certificate, and nobody wants to renew them by hand.
The obvious setup — run certbot on every host — means N rate-limit budgets, N
challenge mechanisms, and N places where renewal can fail silently. So issue in one
place and distribute the result. That part is easy. The hard part is not breaking the
keys along the way.

## The design

**Issuing, on one designated node**

- `certbot` with the **DNS-01** challenge: wildcards work, and nothing has to be
  reachable from the internet on port 80.
- The DNS provider's API credentials live in that node's certbot configuration —
  not in the repository, and not on the consumer nodes.
- `reuse_key = true` in the renewal configuration. This is the load-bearing line:
  private keys stay local and are never re-issued.
- ECDSA keys, one certificate per zone: the apex plus a wildcard
  (`example.com`, `*.example.com`).

**Distributing, through storage that carries only public material**

After a successful renewal the issuer uploads the **fullchain** — certificate and
chain, no private key — to a restic snapshot in object storage that consumer nodes
read.

**Consuming, with a verification step**

Each consumer fetches the fullchain and checks it against its *own* private key
before installing it. If the chain does not match the key, the fetch is refused and
the old certificate stays in place. The comparison is on the **public-key digest** of
each side, not on subject names or serials.

**Seeding private keys once**

A consumer's private key is generated on the issuer and moved to the consumer
directly, over the management channel, once. It never travels through object storage
and never enters version control. After that the key is fixed for the life of the
lineage, and everything else follows from it.

## The traps

None of these is obvious from the documentation, and each one cost time to find.

1. **A committed archive of the certificate directory freezes the keys.** The
   obvious way to seed a new node is to tar up `/etc/letsencrypt` and ship it. That
   archive contains the private keys *as of that day*, and those keys can never be
   replaced — so every later chain fails the digest check on the consumer, which then
   keeps serving the old certificate. Seed keys explicitly and separately; never
   through a snapshot that also carries certificates. If a bootstrap tarball does
   exist, make the code say out loud that its keys may be stale, and reconcile from
   the issuer before the fetch is allowed to install anything.

2. **Renewal rotates the key unless you pin it.** Without `reuse_key = true`,
   `certbot renew` generates a new key pair. The certificate is valid, the issuer is
   happy, and every consumer rejects the chain because its local key no longer
   matches. Renewal must preserve the key, or distribution must carry the key — and
   carrying keys is the thing we are avoiding.

3. **`live/` must stay symlinks.** `certbot` keeps the real files in `archive/` and
   expects `live/<domain>/fullchain.pem` and friends to be symlinks into it. If any
   process replaces those symlinks with real files, renewal stops: certbot reports
   that it could not parse the file and exits with "0 renewals attempted" — a message
   that reads like success.

4. **A failed renewal looks like a healthy host.** When the DNS challenge cannot
   authenticate, the failure is a *renewal* failure: the existing certificate is
   still served, so monitoring that only asks "is HTTPS up?" stays green until the
   certificate expires. Run `renew --dry-run` after any change to credentials, and
   read the first error line rather than the summary.

5. **Credential variables are read by name, and unknown names are ignored.** The DNS
   plugin looks for specific key names. A differently-named or typo'd variable is not
   an error — it is simply not found, and the challenge fails later with a message
   about DNS rather than about configuration.

6. **Do not run the consumer's fetch on the issuer.** The fetch replaces the
   certificate files it manages. On the issuing node those files are certbot's
   symlinks, so a fetch there turns them into real files and triggers trap 3. Exclude
   the issuer from the fetch explicitly — a `when` on the one host, not a comment
   asking people to remember.

## What "done" looks like

Renewal is a non-event, and the verification happens where it matters: the consumer's
fetch parses the staged chain, checks its expiry, and compares its public-key digest
with the local key's before anything is installed. A mismatch is a hard failure, and
the previous certificate keeps serving — so the fleet can be wrong about a
certificate without going down.

A periodic walk is the natural companion: for each host and name, print the not-after
date and the public-key digest of the certificate and of the local key, and let the
one line that does not match stand out.
