# TaxHub SDK

TaxHub works by embedding a complete tax experience directly into your platform, allowing your users to process their taxes without leaving your application.

It's seamless, customizable, and secure — with zero configuration needed.

## Installation

This package is hosted on GitHub Packages and requires authentication.

1. Request an access token from the CTC team
2. Create a `.npmrc` file in your project root:

```ini
@cryptotaxcalculator:registry=https://npm.pkg.github.com
//npm.pkg.github.com/:_authToken=YOUR_CTC_PROVIDED_TOKEN
```

3. Install the package:

```bash
npm install @cryptotaxcalculator/react-sdk
# or
yarn add @cryptotaxcalculator/react-sdk
# or
pnpm add @cryptotaxcalculator/react-sdk
```

## Usage

```tsx
import { TaxHub } from '@cryptotaxcalculator/react-sdk';

function App() {
    return (
        <div>
            <h1>My Crypto App</h1>
            <TaxHub appearance={{ theme: 'cobalt' }} />
        </div>
    );
}
```

The `TaxHub` component will render the TaxHub application by Crypto Tax Calculator. To match the TaxHub to your application see the [Customization](docs/Customization.md) for more information.

## Features

-   Simple drop-in integration
-   Customizable styling (See [Customization](docs/Customization.md))
-   Automated Transaction Import
-   Comprehensive Tax Report Generation

## Requirements

-   React 18 or higher
-   Modern browser support
-   Valid authentication token and theme from CTC team

## Access

This package requires authentication. Please contact the Crypto Tax Calculator team to request access and receive your authentication token.
