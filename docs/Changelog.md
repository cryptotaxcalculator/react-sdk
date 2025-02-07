# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-01-29

### Breaking

-   New `referrerId` property is required. This value is provided by CryptoTaxCalculator and is used to uniquely identify users from this source.

### Added

-   Added language configuration support for internationalization.
-   Added support for referral tracking through First Promoter.
-   Implemented light/dark mode theming support through color mode configuration.
-   Added wallet auto-import functionality through predefined wallet list configuration via the queryParams.

## [0.2.1] - 2025-01-17

### Fixed

-   New environment types preventing developers from installing new package.

## [0.2.0] - 2025-01-16

### Added

-   Added local development support for Tax Hub SDK. Local development requires IP whitelisting - contact Crypto Tax Calculator to get your IP whitelisted

### Fixed

-   Iframe styling caused overflow issue when scaled to 100% width and height of parent.

## [0.1.4] - 2024-12-01

### Added

-   Added custom theming support with advanced styling options
-   Enhanced error handling for authentication failures
