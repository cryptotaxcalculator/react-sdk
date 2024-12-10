# 🚀 CTC React SDK

Simple React components to embed Crypto Tax Calculator in your application.

## 📦 Installation

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

## 🔧 Usage

```tsx
import { TaxCalculator } from "@cryptotaxcalculator/react-sdk";

function App() {
  return (
    <div>
      <h1>My Crypto App</h1>
      <TaxCalculator />
    </div>
  );
}
```

The `TaxCalculator` component will render a full-height iframe containing the Crypto Tax Calculator application.

## ✨ Features

- 🎯 Simple drop-in integration
- 🖼️ Responsive design
- 🔄 Environment-aware URLs
- 🎨 Customizable styling

## 🤝 Requirements

- React 18 or higher
- Modern browser support
- Valid authentication token from CTC team

## 📝 License

MIT © [Crypto Tax Calculator](https://cryptotaxcalculator.io)

## 🔐 Access

This package requires authentication. Please contact the Crypto Tax Calculator team to request access and receive your authentication token.
