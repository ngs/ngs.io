---
title: An MCP Server That Turns Every Flickr API Method into a Tool
slug: "flickr-mcp-server"
description: I've released flickr-mcp-server, an MCP server that makes every Flickr API method available as a tool.
date: "2026-10-22T03:00:00+09:00"
public: true
tags: ["go","oss","mcp","flickr","claude-code"]
archives: ["2026-10"]
alternate: true
# image: main.jpg  # TODO: the main image (992x525) is added by hand
---

I've released [flickr-mcp-server](https://github.com/ngs/flickr-mcp-server), an MCP server that makes every Flickr API method available as a tool.

<!--more-->

## Motivation

I want an LLM to work out the length of a fish from the size of the lure in my fishing photos.

iCloud shared links expire, so they are not practical for handing photos over.

I have a Flickr Pro account.

The existing Flickr MCP implementations don't look like much, so I built my own that offers the full feature set of the Flickr API as tools.

## Usage

A tool is named after its API method with dots replaced by underscores, so `flickr.photos.search` becomes `flickr_photos_search`.

It takes the same arguments as the API and returns Flickr's JSON as is.

Register it with Claude Code like this:

```sh
claude mcp add flickr -- flickr-mcp-server --download-dir ~/Pictures/flickr
```

Without `--max-perms` it starts with `read`, so tools that delete or edit photos are not registered.

With `--download-dir`, `flickr_download` becomes available and saves a photo under that directory at one of the sizes `flickr.photos.getSizes` lists, such as `Medium 800`.

## Installation

With Homebrew:

```sh
brew install ngs/tap/flickr-mcp-server
```

Or with Go 1.26 or later:

```sh
go install go.ngs.io/flickr-mcp-server@latest
```

Create a non-commercial API key in Flickr's [App Garden](https://www.flickr.com/services/apps/create/), then run:

```sh
flickr-mcp-server setup             # save the API key and secret
flickr-mcp-server auth --perms read # allow access in the browser and paste the code
flickr-mcp-server whoami            # check
```

## Under the hood

The tools are not written by hand; they are generated with `go generate` from the method list taken from Flickr's reflection API (`flickr.reflection.getMethods` and `flickr.reflection.getMethodInfo`).

As of October 21, 2026 there are 224 methods, and a token authorized with read permission gives 152 tools.

`live.staticflickr.com`, where the images are served, returns 502 to Go's default User-Agent, so the server sends its own.

## Feedback

Bug reports, feature requests and pull requests are welcome, feel free to send some.

https://github.com/ngs/flickr-mcp-server
