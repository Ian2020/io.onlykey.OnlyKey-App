# README

This is an unofficial flatpak build of the [OnlyKey-App](https://github.com/trustcrypto/OnlyKey-App).

## Table of Contents

<!-- vim-markdown-toc GitLab -->

* [Background](#background)
* [Install](#install)
* [Usage](#usage)
* [Updating](#updating)
* [Outstanding Issues](#outstanding-issues)
* [License](#license)

<!-- vim-markdown-toc -->

## Background

I was keen to get a flatpak for the Onlykey App and it's an outstanding issue on
the app's repo: [Provide a Flatpak as software package for Linux · Issue #105 · trustcrypto/OnlyKey-App](https://github.com/trustcrypto/OnlyKey-App/issues/105)

## Install

## Usage

## Updating

## Outstanding Issues

We had to work around a number of issues to get this flatpak build working. Over
time we might be able to simplify the build if these are addressed.

The first two relate to flatpak's requirement to build without network access.

1. The OnlyKey-App repo does not include a `package-lock.json` which is
   essential for offline builds. If this could be added upstream we would no
   longer need to patch it in the manifest.
2. Nwjs's `npm-installer` supports offline install however it has a bug:
   [#77](https://github.com/nwjs/npm-installer/issues/77). There is a fix but
   it's unreleased. We need nwjs to release this
   [commit](https://github.com/nwjs/npm-installer/commit/86b52bc4dfc3272a6249683abd28a73be5c78eb3#diff-7ae45ad102eab3b6d7e7896acd08c427a9b25b346470d7bc6507b6481575d519)
   and OnlyKey-App to adopt the same release.

   In the meantime we use our own fork: [Ian2020/npm-installer at
   v0.71.1](https://github.com/Ian2020/npm-installer/tree/v0.71.1).
   This forces us to patch `package.json` and `package-lock.json` to point to
   our fork as a git dependency until the issue is resolved upstream.

The next are around the use of the
[flatpak-builder-tools/node](https://github.com/flatpak/flatpak-builder-tools/tree/master/node)
the generate sources for the flatpak manifest.

3. The files `generated-sources.json` and `generated-sources-nw.json` were
   created with
   [flatpak-builder-tools/node](https://github.com/flatpak/flatpak-builder-tools/tree/master/node)
   however this tool has a bug
   [#377](https://github.com/flatpak/flatpak-builder-tools/issues/377) that means
   the wrong list of sources is generated. We have submitted a PR [#378](https://github.com/flatpak/flatpak-builder-tools/pull/378)
   and once accepted we won't have to run a local fork.
4. A second issue with
   [flatpak-builder-tools/node](https://github.com/flatpak/flatpak-builder-tools/tree/master/node)
   is that is does not support the git dependency we use in (2) above. We have a
   fix for that but it's not yet a PR.

The final is around OnlyKey App's build process:

5. The OnlyKey gulp linux release task creates a .deb in a tmp dir and then
   deletes all the artifacts. We have to patch out the deb creation (as it fails
   in our environment) and we need the artifacts to finish building the flatpak.
   Perhaps in future we could have a dedicated flatpak release task in the
   OnlyKey-App repo upstream to rely on.

## License

In accordance with the OnlyKey-App
[license](https://github.com/trustcrypto/OnlyKey-App/blob/master/LICENSE.txt) we
state that:

* OnlyKey-App's code is Copyright (c) 2017, CryptoTrust LLC.
* Flatpaks built with this repo includes software developed by the OnlyKey
  Project [http://www.crp.to/ok](http://www.crp.to/ok).
* We retain and reproduce their copyright notice, conditions and disclaimer
  below:

```text
Copyright (c) 2017, CryptoTrust LLC.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are
met:

1. Redistributions of source code must retain the above copyright
   notice, this list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright
   notice, this list of conditions and the following disclaimer in the
   documentation and/or other materials provided with the distribution.
3. All advertising materials mentioning features or use of this software
   must display the following acknowledgment:
   "This product includes software developed by the OnlyKey Project
   (http://www.crp.to/ok)"
4. The names "OnlyKey" and "OnlyKey Project" must not be used to endorse
   or promote products derived from this software without prior written
   permission. For written permission, please contact admin@crp.to.
5. Products derived from this software may not be called "OnlyKey" nor
   may "OnlyKey" appear in their names without prior written permission
   of the OnlyKey Project.
6. Redistributions of any form whatsoever must retain the following
   acknowledgment:
   "This product includes software developed by the OnlyKey Project
   (http://www.crp.to/ok)"

THIS SOFTWARE IS PROVIDED BY THE OnlyKey PROJECT ``AS IS'' AND ANY
EXPRESSED OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
PURPOSE ARE DISCLAIMED.  IN NO EVENT SHALL THE OnlyKey PROJECT OR ITS
CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
```
