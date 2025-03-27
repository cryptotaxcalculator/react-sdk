# API Reference

## TaxHub Component Props

These properties allow you to configure the TaxHub when it starts up. Changes to these props will not affect the TaxHub if it has already started.

| name                    | type                                                     | description                                                                                                                                                              |
| ----------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `referrerId`            | `string`                                                 | Your provider ID supplied by Crypto Tax Calculator. This is required                                                                                                     |
| `firstPromoterId?`      | `string` (optional)                                      | Your tracking ID supplied by Crypto Tax Calculator.                                                                                                                      |
| `appearance.theme`      | `string`                                                 | A valid theme for the application.                                                                                                                                       |
| `appearance.colorMode?` | `light` \| `dark` (optional)                             | The color mode for the theme                                                                                                                                             |
| `lang?`                 | `'en' \| 'it' \| 'es' \| 'fr' \| 'de'` (optional)        | The language to display the TaxHub in. Defaults to 'en'.                                                                                                                 |
| `queryParams?`          | `Record<string, string \| number>` (optional)            | Optional query params to pass through to the underlying TaxHub.                                                                                                          |
| `initialAuthState`      | `preserve_session` (default) \| `new_session` (optional) | Optional initial authentication state. Determines whether to preserve any existing auth session or start a new session. Defaults to 'preserve_session'.                  |
| `onSetupComplete`       | `() => void` (optional)                                  | Optional callback that fires once after the initial setup is complete. This is useful for any cleanup or side effects that need to run after TaxHub is initially set up. |

## Automatic Imports

### Exchanges

You can provide the TaxHub with a list of exchange integration + api keys + secret and the accounts will be auto-imported for the user after they sign up. 

For example:

```tsx
<TaxHub onIntegrationRequested={(integration) => {
    return [{
        apiKey: 'XXXX',
        secret: 'XXXX'
    }];
}}>
```

### Ethereum L2

You can provide the TaxHub component a list of accounts/networks and the accounts will be auto-imported for the user after they sign up. 

| name       | type     | description                                                             |
| ---------- | -------- | ----------------------------------------------------------------------- |
| `accounts` | `string` | Comma separate list of ethereum wallet addresses e.g. "0x1..23,0x3..45" |
| `networks` | `string` | Comma separated list of chain ids e.g. "1,10"                           |

#### Supported Network Chains

-   81457 Blast
-   7000 Zeta Chain
-   42161 ARB
-   42170 Arbitrum Nova
-   1313161554 Aurora
-   43114 Avalanche
-   8453 Base
-   199 BitTorrent Chain
-   288 Boba
-   56 BSC
-   7700 Canto
-   42220 Celo
-   1024 CLV Chain
-   25 Cronos
-   1 ETH
-   250 Fantom
-   2222 Kava
-   1088 Metis
-   1284 Moonbeam
-   1285 Moonriver
-   10 OPT
-   137 Polygon
-   369 Pulse Chain
-   100 Xdai
-   59144 Linea
-   7777777 Zora
-   5000 Mantle
-   324 ZkSync
-   1101 Polygon ZkEvm
-   534352 Scroll
-   169 Manta Pacific
-   61 Ethereum Classic
-   13371 Immutable
-   14 Flare
-   106 Velas
-   167000 Taiko
-   34443 Mode
-   146 Sonic

## Session Management

The default behaviour is to preserve a session. If you wish to force the start of a new session you can do so using the `initialAuthState` property.

```tsx
import useLocalStorage from '...';

const [shouldCreateNewSession, setShouldCreateNewSession] =
    useLocalStorage(true); // Replace with your own logic

<TaxHub
    initialAuthState={
        shouldCreateNewSession ? 'new_session' : 'preserve_session'
    }
    onSetupComplete={() => {
        // This guarantees if the user refreshes the page,
        // they won't create a new session every time
        setShouldCreateNewSession(false);
    }}
/>;
```

## Development Configuration

Local development requires your IP to be whitelisted. Request your IP to be whitelisted by contacting Crypto Tax Calculator.
After you IP has been whitelisted you can change the TaxHub to "dev" mode which allows you to run it on your local environment.

```tsx
import { TaxHub } from '@cryptotaxcalculator/react-sdk';

function App() {
    return (
        <div>
            <h1>My Crypto App</h1>
            <TaxHub
                referrerId="your-id"
                appearance={{ theme: 'cobalt', colorMode: 'light' }}
                envMode={'dev'}
            />
        </div>
    );
}
``` 