---
title: A URL Shortener on the Cloudflare Workers Free Plan, Managed with Terraform
slug: "terraform-cloudflare-url-shortener"
description: Published terraform-cloudflare-url-shortener, a Terraform module that runs a tiny URL shortener entirely on the Cloudflare Workers free plan.
date: "2026-06-13T00:00:00+09:00"
public: true
tags: ["terraform","cloudflare","workers","url-shortener","oss"]
archives: ["2026-06"]
image: main.jpg
---

I've published [terraform-cloudflare-url-shortener], a Terraform module that
deploys a tiny URL shortener running entirely on the Cloudflare Workers free
plan. Short links are defined declaratively as a Terraform map and baked into
the Worker script — no KV namespace, no database, no paid features required.

The module is available on the [Terraform Registry] as
`ngs/url-shortener/cloudflare`.

<!--more-->

## Motivation

I wanted something like the discontinued
[Google Short Links](https://support.google.com/a/answer/9140741) — the
`go/` links feature where short, memorable slugs redirect to long URLs on a
domain you own.

Most self-hosted URL shorteners are overkill for this. They ship with an
admin UI, a database, and authentication, all to manage what is essentially
a small key-value map that changes a few times a month. Even the "serverless"
ones typically want a Workers KV namespace or a D1 database, which adds
state to provision and another thing to back up.

But if the links live in your Terraform configuration anyway, the data store
becomes unnecessary. The whole link table fits comfortably inside the Worker
script itself, which means:

- **Links are code.** Adding a link is a pull request — reviewed, versioned,
  and revertable through `git log` like everything else.
- **Zero runtime dependencies.** No KV reads, no database round-trips. Every
  request is answered from a constant object in memory at the edge.
- **Free.** Workers scripts, routes, and proxied DNS records are all
  available on the Cloudflare free plan.

## How it works

The module manages just three resources:

1. A **Workers script** rendered from a template, with the `links` map
   embedded as JSON.
2. A proxied **placeholder DNS record** (`AAAA 100::`) for the hostname, so
   traffic reaches Cloudflare's edge. `100::` is the IPv6 discard prefix —
   the record never needs to point at a real origin because the Worker
   intercepts every request.
3. A **Workers route** (`<hostname>/*`) that sends all requests on the
   hostname to the Worker.

Request handling is intentionally boring:

| Request | Response |
|---|---|
| `/<known-slug>` | Redirect to the mapped URL with `redirect_status` (default `301`) |
| `/` | `302` to `root_url`, or `404` when `root_url` is `null` |
| anything else | `404` |

One small but important detail: slug lookups use `Object.hasOwn`, so
prototype property names such as `constructor` or `__proto__` can never be
resolved as links.

Updating links is just `terraform apply` — Terraform re-renders the script
template and uploads the new Worker, and the change is live globally in
seconds.

## Usage

```hcl
module "url_shortener" {
  source  = "ngs/url-shortener/cloudflare"
  version = "~> 0.1"

  account_id = var.cloudflare_account_id
  zone_id    = var.cloudflare_zone_id
  hostname   = "s.example.com"

  root_url = "https://example.com/"

  links = {
    gh      = "https://github.com/ngs"
    resume  = "https://example.com/resume.pdf"
    "a.txt" = "https://example.com/files/a.txt"
  }
}
```

That's the whole setup. With this in place, `https://s.example.com/gh`
redirects to my GitHub profile, and the root path falls back to `root_url`.

The Cloudflare API token used by the provider needs permission to manage
Workers scripts, Workers routes, and DNS records for the target zone. The
module requires Terraform >= 1.0 and the
[cloudflare provider](https://registry.terraform.io/providers/cloudflare/cloudflare/latest)
~> 5.

A few optional knobs:

- `redirect_status` — switch from the default `301` to `302`, `307`, or
  `308` (validated, so a typo fails at plan time).
- `script_name` — override the Workers script name, which defaults to the
  hostname with dots replaced by dashes.
- `root_url` — leave it `null` to serve `404` at the root instead of
  redirecting.

## Testing without credentials

The module ships with two test suites, and neither needs a Cloudflare
account:

```sh
# Terraform native tests (mocked provider, plan-only)
terraform init -backend=false
terraform test

# Worker unit tests (no dependencies, Node.js >= 18)
node --test tests/worker.test.mjs
```

The Terraform tests use the mocked-provider support added in Terraform 1.7
to verify resource wiring at plan time, while the Worker logic is covered by
plain `node --test` with no test framework to install.

The source is on [GitHub][terraform-cloudflare-url-shortener] under the MIT
license. Issues and pull requests are welcome.

[terraform-cloudflare-url-shortener]: https://github.com/ngs/terraform-cloudflare-url-shortener
[Terraform Registry]: https://registry.terraform.io/modules/ngs/url-shortener/cloudflare/latest
