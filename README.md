# 👋 Hi, I'm @maburtobi

**Infrastructure & automation engineer** — I build the machinery that lets a small team
look after a lot of machines, and I keep it boring: repeatable provisioning, TLS that
renews itself, monitoring that says something, and crawlers that stay alive.
*Making infrastructure repeatable, systems faster, and ops less painful.*

---

## 🚀 What I work on

- 🔧 **Fleet configuration as code** — Ansible: one monorepo of roles, host vars and playbooks that provisions and converges every node; drift is caught by re-running, not by hoping
- 🐳 **Containers & orchestration** — Docker, Compose, Kubernetes (RKE2), nginx in front of per-zone TLS, deploys that roll without downtime
- 🔐 **TLS at scale** — Let's Encrypt over the DNS-01 challenge, issued in one place and distributed to consumers, with key pinning so a renewal can never diverge from what the nodes hold; every consumer verifies a fetched chain against its own key before installing it
- 📊 **Observability** — Prometheus with node exporters, and Grafana; dashboards built on the fleet's own data, and monitors that only fire when something really changed
- 🧭 **DNS & network hygiene** — Cloudflare for the public zones, AdGuard Home as the internal resolver, per-host records managed from the same repo as everything else
- 🕸️ **Data collection** — scrapers and ETL into MariaDB, freshness tracked per source, and failure paths that fail loudly instead of writing nothing
- 🤖 **AI operations** — running a fleet of autonomous agents (skills, scheduled jobs, review pipelines) and keeping their output auditable: what changed, why, and how to verify it

---

## 🛠️ Things I maintain in public

- **[tools-and-scripts](https://github.com/maburtobi/tools-and-scripts)** — operator utilities: IP information and TCP port checks, in shell, Go, and uv-runnable Python. Documented for humans and for AI agents (`SKILL.md`), so a tool gets used correctly the first time.
- **Fleet notes** *(in progress)* — architecture writeups from running a self-managed fleet: certificate distribution, crawler freshness, internal DNS, observability. Sanitised: designs and traps, no hostnames or credentials.

Portfolio: **[maburtobi.com](https://maburtobi.com)**

---

## 📫 Let's Connect
- 💬 Got an idea? Need help with something? **DM me / send me an email!**
- 📧 Reach out via [GitHub Discussions](https://github.com/maburtobi/maburtobi/discussions)
- 🔭 Open to infrastructure, platform and DevOps work
- ☕ Buy me a coffee? *(coming soon)*

---

> “On-call rotation: Like musical chairs, but with pagers and no music.”
> – *maburtobi's Midnight Symphony* 🎶📟
