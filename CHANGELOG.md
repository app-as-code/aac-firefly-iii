# Changelog

All notable changes to this deployment definition are documented in this file.
The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Changed

- Documentation (README, icon sources) rewritten in English.

## [0.0.1]

### Added

- Initial App-as-Code service definition for Firefly III core `version-5.5.13`.
- Data Importer `version-0.9.12` as a companion service with its own upload volume.
- MariaDB and Redis dependencies, plus a daily cron container for the static cron endpoint.
- Traefik routing, automatic DNS and an icon for the Homepage dashboard.
