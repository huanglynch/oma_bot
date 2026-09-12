# OMA Bot Desktop

[中文](README.md) · English

A local multi-agent desktop app: concurrent conversations, a Bot workbench, a project rail, and swappable skins.

**Source code is not published.** Official Windows binaries live on [Releases](../../releases) only.

[![Release](https://img.shields.io/badge/Distribution-EXE%20only-c24d1d)](../../releases)
[![License](https://img.shields.io/badge/License-Proprietary-5a5348)](LICENSE)
[![Source](https://img.shields.io/badge/Source-Not%20published-8a8174)](#source-policy)

---

## Download

Open **Releases**, take the latest `OMA-Bot-Setup.exe` or `OMA-Bot-Portable.zip`, and check the SHA256 printed in the release notes before you run it.

Unsigned private software often trips SmartScreen. Trust the publisher, not a random mirror.

---

## What it is

An agent window bound to a local working directory. Conversations are isolated agent instances and can run at the same time.

| Area | Notes |
| --- | --- |
| Concurrent chats | Switching chats does not stop background work |
| Project rail | Pin directories; click to enter, right-click to close |
| Bot workbench | Presets for mail, minutes, editing, translation, planning, research |
| Modes | Fast / default / Polaris |
| Skins | Paper, xuan, celadon, amber, mist, and others |
| Attachments | Drop files on the input |
| Completions | `/` skills, `@` project files |
| Export | Markdown, optional reasoning |

Keys, models, and files stay on your machine.

---

## Requirements

Windows 10/11 x64, 8 GB RAM recommended, and network access to whatever model endpoint you configured.

---

## Source policy

No `.py` in this repository. No source PRs. Redistribute the official Releases URL, not the binary. See [LICENSE](LICENSE).

---

## Publish (maintainers)

Follow [docs/PUBLISH.md](docs/PUBLISH.md). Binaries belong on the Release page, never in Git.
