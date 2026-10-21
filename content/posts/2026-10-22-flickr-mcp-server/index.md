---
title: An MCP Server That Turns Every Flickr API Method into a Tool
slug: "flickr-mcp-server"
description: flickr-mcp-server exposes all 224 Flickr API methods as MCP tools, so an LLM can find your photos by date taken or location and read their EXIF and smaller sizes.
date: "2026-10-22T09:00:00+09:00"
draft: true
public: true
tags: ["go","oss","mcp","flickr","claude-code"]
archives: ["2026-10"]
alternate: true
# image: main.jpg  # TODO: the main image (992x525) is added by hand
---

I released [flickr-mcp-server](https://github.com/ngs/flickr-mcp-server), an MCP server that exposes every Flickr API method as a tool.

From Claude Code or any other MCP client, you can find your own Flickr photos by date taken or location, read their EXIF, and download a smaller size.

<!--more-->

## Motivation

When I go fishing I take a lot of pictures of the fish and the tackle.

I wanted to hand those pictures to an LLM so it could put the catches in order by the time they were taken, or estimate the length of a fish from the lure in the same shot, but getting the pictures to it was the hard part.

I started by passing iCloud shared links, but those links expire, so there was no way to have it look at the same pictures again later.

I already pay for Flickr Pro, so now the pictures go to Flickr and the LLM searches them by date taken or location and fetches only what it needs.

So that it isn't limited to search and download, every method the Flickr API offers is a tool.

## Usage

### Finding photos

Ask Claude Code for "the photos I took on the night of October 16" and it calls `flickr_photos_search` with arguments like these:

```json
{
  "user_id": "me",
  "min_taken_date": "2026-10-16 18:00:00",
  "max_taken_date": "2026-10-16 23:59:59",
  "extras": "date_taken,geo",
  "per_page": "50"
}
```

`me` works because the calls are made as the account you authorized with OAuth, which also means private photos show up in the results.

The tool returns Flickr's JSON as is, and `page` / `pages` / `total` tell it how to fetch the next page.

### EXIF and smaller sizes

Once it has a photo, `flickr_photos_getExif` gives the date taken and camera details, and `flickr_photos_geo_getLocation` gives the location.

`flickr_download` saves the image at one of the sizes `flickr.photos.getSizes` lists (`Large` by default; `Medium 800` is plenty when the LLM only needs to look at it).

Downloads land in `--download-dir` as `<photo id>_<secret>_<size>.<ext>`, and asking for the same photo again returns that file without going back to Flickr.

Nothing is removed automatically: `flickr_cache_status` shows how much the cache holds and `flickr_cache_purge` removes old images.

### The permission ceiling

`--max-perms` (`read`, `write` or `delete`) sets the most the server can ever do.

Methods that need more are not registered as tools, so a server started with `read` doesn't even list a tool that could delete or edit a photo.

With an authorized token, that gives:

| `--max-perms` | Tools |
|---|---|
| `read` | 152 |
| `write` | 215 |
| `delete` | 218 |

Without a token, only the 92 methods that need no login are there.

If your client struggles with that many, narrow it down with `--namespaces photos,photosets,tags`.

Upload and download touch local files, so they only exist when you name a directory with `--upload-dir` or `--download-dir`, and they never read or write outside it.

## Installation

Install it with Homebrew:

```sh
brew install ngs/tap/flickr-mcp-server
```

or with Go 1.26 or later:

```sh
go install go.ngs.io/flickr-mcp-server@latest
```

Create a non-commercial API key in Flickr's [App Garden](https://www.flickr.com/services/apps/create/), save it, and authorize your account:

```sh
flickr-mcp-server setup                 # saves the key and secret to ~/.config/flickr-mcp-server/credentials (0600)
flickr-mcp-server auth --perms read     # open the printed URL, allow access and paste the code Flickr shows
flickr-mcp-server whoami                # check that it worked
```

The permission you ask for in `auth` is the most that token can ever use.

Adding it to Claude Code is one line:

```sh
claude mcp add flickr -- flickr-mcp-server --download-dir ~/Pictures/flickr
```

Note that since May 15, 2025, Flickr has restricted downloads of the original and larger-than-1024px sizes of photos owned by free accounts ([Flickr Blog](https://blog.flickr.net/en/2025/04/15/service-update-original-large-size-download-limitations-on-free-accounts/)).

## Under the hood

It's written in Go with the official [MCP Go SDK](https://github.com/modelcontextprotocol/go-sdk).

The tools aren't written by hand: a list of methods taken from Flickr's own `flickr.reflection.getMethods` and `flickr.reflection.getMethodInfo` (`api/methods.json`) is turned into code by `go generate`.

The same list can also be built from the API documentation pages, and the two I took on October 21, 2026 matched for all 224 methods, down to the required permissions, arguments and descriptions.

CI fails when the generated code doesn't match the list, so keeping up with a new Flickr method is a matter of taking a new snapshot and running `go generate`.

Upload, replace and download aren't REST methods, so those few tools are hand-written.

Authorization is OAuth 1.0a, and instead of running a local HTTP server for the callback, you paste the code the browser shows.

While trying it against the real API, I found that `live.staticflickr.com`, where the images are served from, answers Go's default User-Agent with a 502, so every request now carries a User-Agent with this program's name.

`go test ./...` swaps Flickr for an HTTP stub and never touches the network, and CI fails if coverage drops below 98%.

## Feedback

Bug reports, feature requests and pull requests are all welcome.

https://github.com/ngs/flickr-mcp-server
