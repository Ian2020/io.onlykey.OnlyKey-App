# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [v5.5.0+2]

## Fixed

- [ssh: Could not resolve hostname file: Temporary failure in name resolution](https://github.com/Ian2020/io.onlykey.OnlyKey-App/issues/4)
  The May 2024 update to `org.freedesktop.Sdk.Extension.node18` saw npm upgraded
  from v9 to v10 exposing a problem with our `generated-sources.json`. This
  required a fix to `flatpak-node-generator` in my open PR, here's the
  [commit](https://github.com/flatpak/flatpak-builder-tools/pull/382/commits/9009cbbab5869caf0c911319cc222e5ff38f192e).
- Courtesy of [JamesOsborn-SE](https://github.com/JamesOsborn-SE) the
  app.desktop launcher now specifies the correct version, which refers to the
  desktop spec rather than app itself.

## Changed

- Swap order of the generated-sources. They overwrite each others' patches and we
  more likely to want to see those of OnlyKey-App, so they should come last.
- No longer need to include/download NWJS when we run `flatpak-node-generator`
  as we already have fixed the way it installs itself offline.

## [v5.5.0+1]

### Added

- First release
- OnlyKey v5.5.0

[unreleased]: https://github.com/Ian2020/io.onlykey.OnlyKey-App/compare/v5.5.0%2B2...HEAD
[v5.5.0+1]: https://github.com/Ian2020/io.onlykey.OnlyKey-App/releases/tag/v5.5.0%2B1
[v5.5.0+2]: https://github.com/Ian2020/io.onlykey.OnlyKey-App/releases/tag/v5.5.0%2B2
