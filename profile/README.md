# mgtt — your architecture, executable 

Describe your system once in YAML — components, dependencies, probes, what "healthy" means. mgtt runs the same model in CI (against synthetic scenarios, so drift fails the PR) and at 3am (against the live system, picking the next most discriminating probe). Postmortems come out as scenario patches you merge back into the model.

More on what and why: [mgt-tool/mgtt](https://github.com/mgt-tool/mgtt).

## Repositories

**Core**

- [**mgtt**](https://github.com/mgt-tool/mgtt) — engine, CLI, and MCP server.

**Providers** — each owns one or more component types. Self-contained; install what you use.

- [**mgtt-provider-kubernetes**](https://github.com/mgt-tool/mgtt-provider-kubernetes) — workloads, networking, scaling, storage, RBAC, webhooks, extensibility.
- [**mgtt-provider-aws**](https://github.com/mgt-tool/mgtt-provider-aws) — databases, compute, messaging, storage.
- [**mgtt-provider-docker**](https://github.com/sajonaro/mgtt-provider-docker) — containers and compose stacks.
- [**mgtt-provider-terraform**](https://github.com/mgt-tool/mgtt-provider-terraform) — reasons across the IaC / live-state boundary.
- [**mgtt-provider-tempo**](https://github.com/mgt-tool/mgtt-provider-tempo) — per-span SLO checks over Grafana Tempo.
- [**mgtt-provider-quickwit**](https://github.com/mgt-tool/mgtt-provider-quickwit) — cross-span tracing checks (transaction flows, async hops, consumer pools).

## Getting started

```bash
curl -sSL https://raw.githubusercontent.com/mgt-tool/mgtt/main/install.sh | sh
mgtt init
mgtt model validate
mgtt simulate --all
```

Full walkthrough: [mgt-tool/mgtt](https://github.com/mgt-tool/mgtt#quick-start).
