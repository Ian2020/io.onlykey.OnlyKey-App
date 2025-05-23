This is an unofficial flatpak build of the [OnlyKey-App](https://github.com/trustcrypto/OnlyKey-App).

## Table of Contents

- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [Updating](#updating)
- [Outstanding Issues](#outstanding-issues)
- [Known Issues](#known-issues)
- [License](#license)

## Background

I was keen to get a flatpak for the Onlykey App and it's an outstanding issue on
the app's repo: [#105](https://github.com/trustcrypto/OnlyKey-App/issues/105)

## Install

Prerequistes:

- flatpak-builder
- Base runtime, SDK and node extension from flathub:

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install org.freedesktop.Sdk//23.08 \
                org.freedesktop.Sdk.Extension.node18//23.08 \
                org.freedesktop.Platform//23.08
```

Steps:

- Checkout this repo at the desired tag for the version of OnlyKey-App you want
  to run, e.g. v5.5.0+1 for OnlyKey-App v5.5.0. The `+N` suffix allow us to
  give this repo it own version number but still tie it to OnlyKey's version.

- Run:

```bash
flatpak-builder --install --user --force-clean build io.onlykey.OnlyKey-App.yaml
```

You might see this:

```bash
Failed to download sources: module onlykey-app: Transferred a partial file
```

Just re-run the command, it's some kind of rate-limit on downloading
dependencies for the build I think. Each time you run a build we use a cache
so you won't have to re-download things that were already successfully obtained.

## Usage

The OnlyKey-App should be installed on your system and available as a desktop
app with icon. You can also run it from the command line with:

```bash
flatpak run io.onlykey.OnlyKey-App
```

Note you still need to follow the other Linux install instructions to get your
device working in full: [Using OnlyKey with
Linux](https://docs.onlykey.io/linux.html). In particular make sure you have
setup the UDEV rule.

## Updating

These instructions are for updating the version of OnlyKey-App in the flatpak
built by this repo.

Prerequisites:

- Node and npm
- `flatpak-node-generator` from [flatpak-builder-tools/node](https://github.com/flatpak/flatpak-builder-tools/tree/master/node).
  My PR (which adds support for git sources) is required:
  [\[node\] support git sources #382](https://github.com/flatpak/flatpak-builder-tools/pull/382).
  Grab it with:  `git clone -b gitrefs git@github.com:Ian2020/flatpak-builder-tools.git`
- Before you start make a note of what version of OnlyKey-App you're building
  and what version of NWJS it uses.

Steps:

- Checkout [trustcrypto/OnlyKey-App](https://github.com/trustcrypto/OnlyKey-App/)
  at the relevant tag for the new version you want to build.

  - If we still need to use our npm-installer fork (see outstanding issues
    below) we need to replace the nw dependency with the correct commit or of
    our fork. This depends on the version of NWJS needed and work may required
    in this repo first to bring it up to date also. Here's the right command for
    NWJS 0.71.1:

    ```bash
    npm uninstall nw
    npm install git+https://git@github.com/Ian2020/npm-installer.git#8e1575a4abf16901924a6146835334ddeaa17b14
    ```

  - If there is no `package-lock.json` in the repo (an outstanding issue below)
    run `npm install` to generate one (the above step may have already generated
    it though).

  - Run `flatpak-node-generator npm package-lock.json`. The
    flatpak-node-generator project uses poetry so install poetry if needed
    also. You can then run `poetry install` and use the resulting `.venv`
    virtual environment to run it.

  - Take the resulting file `generated-sources.json` and copy into your working dir
    of this repo.

    - Replace any occurrences of `FLATPAK_BUILDER_BUILDDIR/package` with
      `FLATPAK_BUILDER_BUILDDIR/onlykey-app/package`.

  - Create patch files for `package.json` and `package-lock.json` using `git diff package.json | cat` or similar and copy into the working dir of this
    repo (replacing `package.json.patch` and `package-lock.json.patch`).

    - Replace any occurrences in those files of `/package` with `/onlykey-app/package`.

- If we still need to use our npm-installer fork (see outstanding issues below)
  then we also need to generate sources for it. This is because as a git
  dependency of OnlyKey-App npm will have to build it first and therefore we
  need its sources too. Checkout the branch of our fork that matches the
  required version of NWJS that OnlyKey-App uses e.g.
  [v0.71.1](https://github.com/Ian2020/npm-installer/tree/v0.71.1). If
  a version is missing then raise an issue in the repo
  [here](https://github.com/Ian2020/npm-installer/issues).

  - Inside the working dir run `flatpak-node-generator npm package-lock.json`
  - Take the resulting file `generated-sources.json` and copy into your working dir
    for this repo but rename it `generated-sources-nw.json`

- Update the version of the NWJS archive we download from
  [https://dl.nwjs.io](https://dl.nwjs.io) in the manifest:
  `io.onlykey.OnlyKey-App.yaml`

- Update the tag of the OnlyKey-App repo we use in source
  in the manifest and anywhere else the version number appears in there:
  `io.onlykey.OnlyKey-App.yaml`

- Now run the steps from the "Install" section above and hope for a successful
  build.

- Test the app.

- Update the CHANGELOG and commit your changes and tag with the version of
  OnlyKey-App and a numbered suffix for our own versioning e.g. v5.5.0+1.

  ```bash
  git tag v5.5.0+1
  git push
  git push --tags
  ```

  Make a release on GitHub's website.

## Outstanding Issues

We had to work around a number of issues to get this flatpak build working. Over
time we might be able to simplify the build if these are addressed.

The first two relate to flatpak's requirement to build without network access.

1. The OnlyKey-App repo does not include a `package-lock.json` which is
   essential for offline builds. If this could be added upstream we would no
   longer need to patch it in the manifest.

1. Nwjs's `npm-installer` supports offline install however it has a bug:
   [#77](https://github.com/nwjs/npm-installer/issues/77). There is a fix but
   it's unreleased. We need nwjs to release this
   [commit](https://github.com/nwjs/npm-installer/commit/86b52bc4dfc3272a6249683abd28a73be5c78eb3#diff-7ae45ad102eab3b6d7e7896acd08c427a9b25b346470d7bc6507b6481575d519)
   and OnlyKey-App to adopt the same release.

   In the meantime we use our own fork: [Ian2020/npm-installer at
   v0.71.1](https://github.com/Ian2020/npm-installer/tree/v0.71.1).
   This forces us to patch `package.json` and `package-lock.json` to point to
   our fork as an npm git dependency until the issue is resolved upstream.

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
1. A second issue with
   [flatpak-builder-tools/node](https://github.com/flatpak/flatpak-builder-tools/tree/master/node)
   is that is does not support the git dependency we use in (2) above. We have a
   fix for that: [#382](https://github.com/flatpak/flatpak-builder-tools/pull/382)

The final minor issue is around OnlyKey App's build process:

5. The OnlyKey gulp linux release task creates a .deb in a tmp dir and then
   deletes all the artifacts. We need to skip the deb creation (as it fails in
   our flatpak environment) and we need the artifacts to finish building the
   flatpak. Perhaps in future we could have a dedicated flatpak release task in
   the OnlyKey-App repo upstream.

For the future:

- We're not really using `flatpak-node-generator`'s nwjs support, by which it
  adds a couple extra sources to download and extract nwjs into a known
  location in `generated-sources.json`.
- We are not dealing with different architectures
- We are not dealing with different locales
- Do we have a truly minimal flatpak?

## Known Issues

- The build produces this message:

  ```log
  ln: failed to create symbolic link 'flatpak-node/cache/node-gyp/18.18.1/include': File exists
  ```

  ...but it seems to have no ill effect.

- NPM issues some warnings but these need to be addressed upstream:

  ```log
  npm WARN config production Use `--omit=dev` instead.
  npm WARN deprecated har-validator@5.1.5: this library is no longer supported
  npm WARN deprecated uuid@3.4.0: Please upgrade  to version 7 or higher. Older versions may use Math.random() in certain circumstances, which is known to be problematic. See https://v8.dev/blog/math-random for details.
  npm WARN deprecated request@2.88.2: request has been deprecated, see https://github.com/request/request/issues/3142
  ```

  ...but it still seems to work.

## License

In accordance with the OnlyKey-App
[license](https://github.com/trustcrypto/OnlyKey-App/blob/master/LICENSE.txt) we
state that:

- OnlyKey-App's code is Copyright (c) 2017, CryptoTrust LLC.
- Flatpaks built with this repo includes software developed by the OnlyKey
  Project [http://www.crp.to/ok](http://www.crp.to/ok).
- We retain and reproduce their copyright notice, conditions and disclaimer
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
