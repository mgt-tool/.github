<img src="https://raw.githubusercontent.com/mgt-tool/mgtt/main/docs/images/mgtt_icon_large.png" align="left" width="140" alt="mgtt">

# mgtt — your architecture, executable

Your architecture diagram is accurate the day it is drawn and lies a little more every day after. So when something breaks at 3am — and the person who built the system is asleep — you open three terminals and start guessing.

`mgtt` makes the diagram executable. You describe the system once in one YAML model — the components, the arrows between them, the probes that decide what *healthy* means for each — and the same engine runs that model two ways: as a test in CI, and as your incident responder at 3am. One model both times, so it cannot quietly drift out of sync with production.

<br clear="left">

## What that looks like

In CI, the scenarios run on every PR — a broken component here, a healthy baseline there, a partial degradation in between:

```console
$ mgtt simulate --all

  all components healthy                   ✓ passed
  api degraded, rds cold-cache             ✓ passed
  edge throttled, downstream healthy       ✓ passed

  3/3 scenarios passed
```

Every scenario is a test of the model's reasoning, including *"everything is fine"* when nothing is broken. Rename `rds` and forget the thing that depends on it, and the PR that broke the model never merges.

At 3am, the same engine runs against the live system, real probes standing in for the synthetic facts:

```console
$ mgtt diagnose --suspect api

  ▶ probe nginx upstream_count       ✗ unhealthy
  ▶ probe api ready_replicas         ✗ unhealthy
  ▶ probe rds available              ✓ healthy  ← eliminated
  ▶ probe frontend ready_replicas    ✓ healthy  ← eliminated

  Root cause: api.degraded
  Chain:      nginx ← api
  Probes run: 4
```

It names the broken component, eliminates the healthy ones, and hands back the chain from symptom to cause. Four probes — each picked for how much it discriminates — instead of a Slack thread and a thousand educated guesses about which `kubectl` to try next. Partial visibility (an RBAC refusal, a transient throttle) surfaces as a flag on the verdict rather than an abort.

Afterwards, `mgtt incident end --suggest-scenarios` writes the failure you just fought through back out as a scenario patch. Merge it, and the engine has to diagnose that situation correctly forever: postmortems become regression tests, and what one engineer knew about how the system fails now lives in version control.

## Repositories

**Core**

- [**mgtt**](https://github.com/mgt-tool/mgtt) — the model schema, the constraint engine, the CLI, and the MCP server that lets an agent drive the same diagnosis loop a human does.

**Providers** — the adapter layer: each owns one or more component types and turns the engine's intent into backend-specific commands. Credentials live here and never reach the engine. Self-contained, so you install only the ones you use.

- [**mgtt-provider-kubernetes**](https://github.com/mgt-tool/mgtt-provider-kubernetes) — 37 types, from Deployment and Ingress out to CRDs, webhooks, RBAC and storage, probed through `kubectl`.
- [**mgtt-provider-aws**](https://github.com/mgt-tool/mgtt-provider-aws) — 14 types across compute, data and edge — RDS, ElastiCache, MQ, S3, CloudFront, EKS, VPC, IAM, ACM, ECR, SSM — probed through the AWS CLI.
- [**mgtt-provider-docker**](https://github.com/mgt-tool/mgtt-provider-docker) — containers and Compose stacks, for the laptop and the single-host deployment.
- [**mgtt-provider-terraform**](https://github.com/mgt-tool/mgtt-provider-terraform) — observes Terraform-managed infrastructure, so the engine can reason across the IaC / live-state boundary.
- [**mgtt-provider-tempo**](https://github.com/mgt-tool/mgtt-provider-tempo) — per-span SLO checks (p99, breach duration, error rate) over Grafana Tempo's TraceQL Metrics.
- [**mgtt-provider-quickwit**](https://github.com/mgt-tool/mgtt-provider-quickwit) — cross-span tracing checks: transaction flows, async hops, consumer pools.

## Getting started

```bash
curl -sSL https://raw.githubusercontent.com/mgt-tool/mgtt/main/install.sh | sh

mgtt init             # scaffold system.model.yaml
mgtt model validate   # check the model
mgtt simulate --all   # run the scenarios
```

Then [the quick start](https://github.com/mgt-tool/mgtt/blob/main/docs/getting-started/quickstart.md) — end to end in five minutes — or the [blue/green storefront](https://github.com/mgt-tool/mgtt/blob/main/docs/examples/blue-green-storefront.md), a twenty-component real system with five scenarios and the lessons from running them. Reference, provider catalogue and specs live at [mgt-tool.github.io/mgtt](https://mgt-tool.github.io/mgtt/).

## Where it does not belong

`mgtt` names the cause and the chain, then stops: it never takes the fix action for you. There is no dashboard, because the reasoning has to be right before anything is worth drawing. And it checks a system that is running, not one that is being designed — TLA+ checks your design; `mgtt` checks your production.
