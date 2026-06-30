---
title: "Modelmap Analyzer Update"
slug: "modelmap-analyzer"
description: An update to Modelmap Analyzer — SVG export support, more stable rendering, the new "Recenter on selection" and "Fit to screen" actions, and Fluent 2 conformance.
date: "2026-07-01T05:00:00+09:00"
public: true
tags: ["modelmap","modelmap-analyzer","excel","google-sheets"]
archives: ["2026-07"]
image: main.png
---

A long-overdue update to **Modelmap Analyzer**, which I've been working on since 2019.

<!--more-->

## What Modelmap Analyzer is

It's an add-in for Google Sheets and Excel on Office 365 that analyzes which cells a formula refers to and shows you those dependencies as a tree.

[Product site](https://analyzer.modelmap.co/)

I built this software at Modelmap, a startup I joined in 2019. When that company wound down, the product moved to instance0, inc. in 2024, which has kept it published and on sale since.

References:
- [Blog post from when I joined (Japanese)](https://ja.ngs.io/2019/02/12/modelmap/)
- [Release post on the transfer (Japanese)](https://ins0.jp/news/modelmap-analyzer/)

This time I've made two feature updates and a price change to Modelmap Analyzer.

## SVG export support

Until now you could only download the dependency tree as a **PDF**. Now you can **export it as SVG** too.

I also reworked the rendering, so exports are more stable and large trees hold together better.

SVG stays crisp no matter how far you zoom in, so it's easy to drop into specs and review docs, or print out to look over.

## Improved UI stability

- Added **Recenter on selection** and **Fit to screen** for arranging the tree view
- Updated the UI to conform to **[Fluent 2](https://fluent2.microsoft.design/)**
- Added light and dark mode switching

## Price change

I revisited the pricing and dropped it to **US$4 per month, billed annually**.

## How to use it

- **Excel / Microsoft 365**: [Microsoft AppSource](https://appsource.microsoft.com/product/office/WA200006896)
- **Google Sheets**: [Google Workspace Marketplace](https://workspace.google.com/marketplace/app/modelmap_analyzer/239836082546)

## Feedback

If you hit a bug or have a feature request, please reach out via the [support page](https://analyzer.modelmap.co/support).
