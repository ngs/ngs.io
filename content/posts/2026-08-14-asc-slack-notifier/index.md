---
title: "asc-slack-notifier: App Store Connect Review Status in Slack"
slug: "asc-slack-notifier"
description: Introducing asc-slack-notifier, a Go server that receives App Store Connect webhooks and posts review status and build events to Slack. One binary that runs on Cloud Run or AWS Lambda, with optional App Store Connect API enrichment.
date: "2026-08-14T09:00:00+09:00"
public: true
tags: ["asc-slack-notifier","app-store-connect","slack","go","ios","oss"]
archives: ["2026-08"]
---

**[asc-slack-notifier](https://github.com/ngs/asc-slack-notifier)** is a server I built that receives [App Store Connect webhooks](https://developer.apple.com/documentation/appstoreconnectapi/webhooks) and posts review status and build events to Slack.

<!--more-->

## Motivation

I've been on release duty for an iOS app at work. Every time we submit, the team wants to know in Slack when the app goes in review, gets approved, or gets rejected — but App Store Connect notifies the individual account by email and push, not the team. So I ended up being the person who opens App Store Connect and pastes screenshots into the channel every time the state moves.

App Store Connect now offers webhooks, so I wrote a small relay that turns them into Slack messages.

Once I started, it turned out the webhook payload is bare: a version state change event carries **only the resource UUID and the old and new states**. The payload alone cannot tell you which app or which version moved, which makes a shared channel useless the moment you have more than one app. So I added optional enrichment — give it an App Store Connect API key and it looks the resource up before posting, adding the app name, version and build number, plus an "Open in App Store Connect" button.

## Usage

A webhook arrives and turns into a Block Kit message with an emoji per state (`READY_FOR_REVIEW` 📝, `PENDING_APPLE_RELEASE` ⏳, `READY_FOR_DISTRIBUTION` ✅, `REJECTED` ❌, …). Event types Apple hasn't documented yet are still delivered with a generic key/value rendering, so notifications are never silently dropped.

The Slack destination is either an Incoming Webhook URL or a bot token plus channel (`chat.postMessage`).

With an App Store Connect API key configured (read-only access with the `Developer` or `App Manager` role is enough), version state and build notifications gain **App / Version / Build** fields and a button that jumps straight to the app's distribution page or TestFlight page. Enrichment is strictly optional: if the API is down at delivery time, the message still goes out un-enriched.

## Setup

The quickest route is to fork the repository and let GitHub Actions deploy it: set the `DEPLOY_TARGET` repository variable to `cloudrun` or `lambda`, add the secrets, and every push to master/main deploys your instance. [docs/DEPLOYMENT.md](https://github.com/ngs/asc-slack-notifier/blob/master/docs/DEPLOYMENT.md) walks through it.

Deploying to Cloud Run by hand is just:

```sh
gcloud builds submit --tag "$IMAGE"
gcloud run deploy asc-slack-notifier \
  --image "$IMAGE" \
  --region asia-northeast1 \
  --allow-unauthenticated \
  --set-secrets "ASC_WEBHOOK_SECRET=asc-webhook-secret:latest,SLACK_WEBHOOK_URL=slack-webhook-url:latest"
```

Then register the webhook with the App Store Connect API:

```sh
curl -sS -X POST 'https://api.appstoreconnect.apple.com/v1/webhooks' \
  -H "Authorization: Bearer $TOKEN" \
  -H 'Content-Type: application/json' \
  -d '{
    "data": {
      "type": "webhooks",
      "attributes": {
        "name": "Slack notifier",
        "url": "https://your-service.example.com/webhook",
        "secret": "your-webhook-secret",
        "enabled": true,
        "eventTypes": [
          "APP_STORE_VERSION_APP_VERSION_STATE_UPDATED",
          "BUILD_UPLOAD_STATE_UPDATED",
          "BUILD_BETA_DETAIL_EXTERNAL_BUILD_STATE_UPDATED"
        ]
      },
      "relationships": {
        "app": { "data": { "type": "apps", "id": "'"$APP_ID"'" } }
      }
    }
  }'
```

One gotcha: the webhook `secret` is **not something Apple issues — you make it up yourself**. Generate a value with something like `openssl rand -hex 32` and give the identical string to both sides: `attributes.secret` when registering the webhook, and `ASC_WEBHOOK_SECRET` on the server. If the two drift apart, every delivery is rejected with `401`.

The API private key can be passed as a path to the `.p8` file, as raw PEM contents, or base64-encoded — the same convention as fastlane's `key_content`, so platforms that only carry string secrets are fine.

## Under the hood

It's a single Go binary. Depending on one environment variable it runs as a plain HTTP server (Cloud Run, Kubernetes, anywhere) or behind AWS API Gateway on Lambda.

Every delivery is authenticated before anything else: the `x-apple-signature` HMAC-SHA256 header is verified against the raw request body, compared in constant time. When Slack can't be reached the service returns `502`, so App Store Connect redelivers instead of dropping the event.

## Feedback welcome

I built this as my own release-duty tool, but it should be useful for any team shipping iOS apps. It's published under the [MIT license](https://github.com/ngs/asc-slack-notifier/blob/master/LICENSE).

Problem reports and feature requests — including how a particular event type should be rendered — are welcome via [GitHub Issues](https://github.com/ngs/asc-slack-notifier/issues) or pull requests.
