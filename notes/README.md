# Fleet notes

Writeups from running a self-managed fleet: the designs that work, and the traps
that cost real time. Sanitised — no hostnames, no addresses, no credentials, no
client names.

1. [Distributing Let's Encrypt certificates to a fleet](tls-at-scale.md)
2. Keeping a crawler fleet honest *(in progress)* — freshness tracked per source,
   and failure paths that exit non-zero instead of quietly writing nothing
3. DNS as a service for the internal network *(in progress)* — one resolver pair,
   per-host records managed from the same repository as everything else
4. Running an agent fleet in production *(in progress)* — skills, scheduled jobs,
   review pipelines, and keeping autonomous changes auditable

## Why write these down

Every trap in these notes was found the expensive way, by a check that failed.
They exist so the next person — including future me — spends five minutes on it
instead of five hours.
