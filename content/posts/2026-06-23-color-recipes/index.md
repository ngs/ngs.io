---
title: "color.recipes: A Simple Color-Scheme Collection"
slug: "color-recipes"
description: Published color.recipes, a simple color-scheme collection you can filter by tag and download or copy in various formats. Missing combinations can be added yourself via a PR.
date: "2026-06-23T09:00:00+09:00"
public: true
tags: ["color-recipes","color","design","oss"]
archives: ["2026-06"]
image: main.png
---

I've published **[color.recipes](https://color.recipes)**, a color-scheme collection you can filter by tag.

Schemes you like can be downloaded or copied in various formats.

The schemes are managed in a public GitHub repository, so any combination that's missing you can make yourself and add via a PR.

<!--more-->

## Motivation

There are already several similar color services out there.

But they were all loaded with extra features or showed ads, and none of them felt right to me.

I started building this software aiming for a simple color-scheme collection that suits me and is pleasant to use.

## How to use it

- **Filter by tag.** Pick more than one and you'll only see schemes that match all of them (e.g. both `winter` and `city`).
- Schemes you like can be **downloaded or copied in various formats** — CSS, SCSS, Tailwind, JSON, SVG, Swift, and more.
- Each scheme has a **permalink**. Share it and it previews with that scheme's thumbnail (an OpenGraph image).

### Missing a combination? Make it yourself and PR

When a search returns nothing, you can build a scheme and add it right there.

![The prompt to generate a scheme, plus the JSON paste and submit form, shown when a search has no matches](compose.png)

1. Copy the **AI prompt** the page shows you
2. Paste it into your own generative AI and have it produce scheme JSON
3. Paste the JSON back and **preview it in place**
4. Log in with GitHub and **choose a fork owner** (your account or an org)
5. One button does **fork → commit → PR**

Scheme data is just `schemes/*.json` in the repo, so a contribution is "a PR that adds one JSON file."

## Feedback welcome

Feedback is handled through GitHub Issues — if you hit a bug or notice something, please file one.

And please send **PRs to add new color schemes**. The repository is here:

**https://github.com/ngs/color.recipes**
