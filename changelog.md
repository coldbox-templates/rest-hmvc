# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

* * *

## [Unreleased]

## [7.5.0] - 2025-03-19

## [7.4.0] - 2024-05-10

### Added

- Mementifier config additions were missing
- Updated to use latest dev dependencies

## [7.3.0] - 2023-06-14

### Added

- Vscode mappings for cbsecurity
- Tons of inline docs for module configurations so newbies can find what they need
- Leveraging the auth `User` of the `cbsecurity` module, so we can reuse what's already been built.

### Fixed

- Gitignore updates so it doesn't ignore 'config/modules'

## [7.2.0] - 2023-05-19

### Fixed

- Added `allowPublicKeyRetrieval=true` to the `db` connection string
- Added missing bundle name and version in `.cfconfig.json` and `.env.example`

## [6.15.0] => 2023-MAR-20

### Added

- Changelog Tracking
- Github actions for auto building
- Latest ColdBox standards
- UI Updates
- Latest Alpine + Bootstrap Combo
- vscode introspection and helpers
- Docker build and compose consolidation to the `build` folder
- Cleanup of `tests` to new standards
- Added routing conventions to make it easier for the cli to add routes.

[unreleased]: https://github.com/coldbox-templates/rest-hmvc/compare/v7.5.0...HEAD
[7.5.0]: https://github.com/coldbox-templates/rest-hmvc/compare/v7.4.0...v7.5.0
[7.4.0]: https://github.com/coldbox-templates/rest-hmvc/compare/v7.3.0...v7.4.0
[7.3.0]: https://github.com/coldbox-templates/rest-hmvc/compare/d9eb531a5efc9aa3d92fce4736f643d1cde16c0a...v7.3.0
