# TaxHub SDK

TaxHub works by embedding a complete tax experience directly into your platform, allowing your users to generate tax reports without leaving your application.

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

The `TaxHub` component will render the TaxHub application by Crypto Tax Calculator. To match the TaxHub to your application see the [Customization](/Customization) for more information.

### React Props

| name                    | type                                   | description                                                     |
| ----------------------- | -------------------------------------- | --------------------------------------------------------------- |
| `appearance.theme`      | `string`                               | A valid theme for the application.                              |
| `appearance.colorMode?` | `light` \| `dark`                      | The color mode for the theme                                    |
| `lang`                  | `'en' \| 'it' \| 'es' \| 'fr' \| 'de'` | The language to display the TaxHub in. Defaults to 'en'.        |
| `queryParams`           | `Record<string, string \| number>`     | Optional query params to pass through to the underlying TaxHub. |

## Features

### Simplified Integration

-   **Drop-In components**: Add the tax calculator to your app in minutes.
-   **Responsive design**: Optimized for mobile, tablet, and desktop.

### Advanced Customization

-   **Theming options**: Customize colors, fonts, and border styles to align with your branding.
-   **Dynamic styling**: Match the tax centre to your app’s look and feel effortlessly.

For further details see [Customization](/Customization)

### Developer-friendly

-   **Environment-aware URLs**: Adapts automatically for development, staging, and production environments.
-   **Clear documentation**: Guides and code examples to ensure a smooth development experience.

## Requirements

-   React 18 or higher
-   Modern browser support

## Deploying TaxHub

To deploy the TaxHub, request Crypto Tax Calculator to whitelist your domain.
For example, if you were deploying at `taxhub.example.com`, Crypto Tax Calculator will whitelist `taxhub.example.com` and then the TaxHub will appear correctly at that domain.

## Local Development

Local development requires your IP to be whitelisted. Request your IP to be whitelisted by contacting Crypto Tax Calculator.
After you IP has been whitelisted you can change the TaxHub to "dev" mode which allows you to run it on your local environment.

```tsx
import { TaxHub } from '@cryptotaxcalculator/react-sdk';

function App() {
    return (
        <div>
            <h1>My Crypto App</h1>
            <TaxHub
                appearance={{ theme: 'cobalt', colorMode: 'light' }}
                envMode={'dev'}
            />
        </div>
    );
}
```

## FAQ

-   Q: How do I request access to the SDK?

    A: Contact the CTC team to obtain your authentication token and theme configuration.

-   Q: Is the SDK updated regularly?

    A: Absolutely. The SDK automatically pushes updates for regulatory changes and new features without requiring changes to your core codebase.

-   Q: How long does it take to integrate?

    A: Most integrations are completed in less than a day.

## Support

For questions or assistance, reach out to us:

-   Live Chat via cryptotaxcalculator.io
-   Email via info@cryptotaxcalculator.io.
