---
title: r7crypto — R7 Crypto Lab
description: An interactive cryptography workbench in Rust — learn, try, verify, make it visible
tags:
  - project
  - open-source
  - security
  - cryptography
---

# R7 Crypto Lab (`r7crypto`)

> **Learn · Try · Verify · Make it visible**

An interactive cryptography workbench in Rust. The core thesis is still entropy — *"looks random" is not enough* — but the scope is wider: randomness, hashes, ciphers, signatures, and later system-level (blockchain) and post-quantum reading.

> [!info] Links
> - Source, releases and the live web build: [github.com/rvoidex7/r7crypto](https://github.com/rvoidex7/r7crypto)

## Why it exists

Four benefits, one handle:

| Benefit | What it means |
|---|---|
| **Learn** | A concept becomes something you can change and watch; the app teaches the material back. |
| **Try** | A reader follows a link from these notes into the browser app and experiments without installing anything. |
| **Verify** | Claims are checkable: golden vectors, adversarial inputs, headless UI tests and CI. |
| **Make it visible** | The experiment, the evidence and the notes stay connected under one name, so a claim can be followed from here to the running app and back. |

The fourth benefit is about checkability, not a personal portfolio claim. The work is public; anyone can inspect it. For what is shipped and what is still open, see [[Projects/r7crypto/Roadmap|Roadmap]].

## Status

| | |
|---|---|
| Platforms | Linux, Windows, browser/WASM — one codebase |
| GUI | `egui` / `eframe`, with screen-reader support via `accesskit` |
| Tests | 197, plus the feature matrix, wasm type-checks and a headless browser smoke test |
| In progress | Avalanche distribution, benchmark labelling, Two-Time Pad, in-app links to these notes |
| Deferred | Android / Play Store, macOS, aarch64, mobile tab bar |

Current release notes and binaries live in the [repository releases](https://github.com/rvoidex7/r7crypto/releases); they are not duplicated here.

## In this folder

- [[Projects/r7crypto/Architecture|Architecture]] — module layout, data flow, design decisions
- [[Projects/r7crypto/Dependencies|Dependencies]] — every crate and why it is there
- [[Projects/r7crypto/Interface|Interface]] — what the tabs do today
- [[Projects/r7crypto/Build-and-Release|Build and Release]] — local build, CI, packaging
- [[Projects/r7crypto/Roadmap|Roadmap]] — current priority order and where to contribute
