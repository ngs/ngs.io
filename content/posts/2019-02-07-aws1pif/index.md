---
title: Import AWS credentials.csv to 1Password using aws1pif
description: Open-sourced aws1pif, a command line tool that converts AWS credentials.csv to 1Password .1pif file format
date: "2019-02-07T21:00:00+09:00"
public: true
tags: ["aws","cli","tool","golang"]
alternate: true
archives: ["2019-02"]
image: import.jpg
---
![](desktop.jpg)

I've just open-sourced **aws1pif**, a command line tool that converts AWS credentials.csv to 1Password .1pif file format

[ngs/aws1pif](https://github.com/ngs/aws1pif)

<!--more-->

## Install

You can install aws1pif using Homebrew or `go install` command

```sh
brew install ngs/formulae/aws1pif
```

or

```sh
go install github.com/ngs/aws1pif
```

## How to Use

`aws1pif` converts CSV standard input to .1pif JSON file format as standard output.

```sh
cat ~/Downloads/credentials.csv | aws1pif > aws.1pif
```

1Password will launch when opening .1pif file and show confirmation dialog

![](import.jpg)

Currently CSV with multiple credentials is not supported.
