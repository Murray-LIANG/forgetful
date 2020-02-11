---
title: "Pip Download and Install Pandas"
date: 2020-02-10T16:37:31+08:00
lastmod: 2020-02-10T16:37:31+08:00
draft: false
show_in_homepage: true
description_as_summary: true
license: ""

tags: [
    "pip",
    "pandas",
]

comment: true
toc: true
auto_collapse_toc: true
---
# Pip Download and Install

```console
$ # Download
$ pip download --platform win_amd64 --only-binary=:all: --python-version 37 --implementation cp --abi cp37m pandas

$ # Install
$ pip install --no-index --find-links=. pandas

```