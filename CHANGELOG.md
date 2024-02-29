# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## \[Unreleased\]

## Fixed

- Courtesy of [JamesOsborn-SE](https://github.com/JamesOsborn-SE) the
  app.desktop launcher now specifies the correct version, which refers to the
  desktop spec rather than app itself.

## Changed

- Swap order of the generated-sources. They overwrite each others' patches and we
  more likely to want to see those of OnlyKey-App, so they should come last.
- No longer need to include/download NWJS when we run `flatpak-node-generator`
  as we already have fixed the way it installs itself offline.

## \[v5.5.0+1\]

### Added

- First release
- OnlyKey v5.5.0
