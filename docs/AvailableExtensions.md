# Available Extensions

This document details optional extensions that can be added to the core CryptoTaxCalculator integration (e.g., via the TaxHub SDK or a direct partnership) to enable enhanced data sharing and functionality between the partner platform and CTC.

---

## Common API Authentication (HMAC-SHA256)

Unless otherwise specified (like for incoming webhooks), API endpoints provided by CTC for partner data access require authenticated requests using HMAC-SHA256 signatures. This ensures that requests originate from a trusted partner.

**Credentials:** You will be provided with a unique Partner **API Key** and **API Secret** by CTC during onboarding.

**Mechanism:**

1.  **Headers Required:** All authenticated API requests MUST include the following headers:
    - `X-API-Key`: Your unique API Key.
    - `X-Timestamp`: The current Unix timestamp (seconds since epoch). Requests with timestamps significantly skewed from server time may be rejected to prevent replay attacks.
    - `X-Signature`: The Base64 encoded HMAC-SHA256 signature calculated as described below.
2.  **Signature Generation:**
    - **String to Sign:** Concatenate the following elements using a newline character (`\n`) as a separator:
      - Unix Timestamp (from `X-Timestamp` header)
      - HTTP Method (e.g., `GET`, `POST`)
      - Request Path (The full path, including any resource IDs, e.g., `/v1/partner/cost_basis/transactions/user123`)
      - Query String (Alphabetically sorted by parameter name, e.g., `asset_ticker=BTC&limit=100`). If no query string, use an empty string.
      - Request Body (The raw request body string). If no body (e.g., for GET requests), use an empty string.
    - **Compute HMAC:** Calculate the HMAC-SHA256 hash of the "String to Sign" using your **API Secret** as the key.
    - **Encode:** Base64 encode the resulting binary hash. This encoded string is the value for the `X-Signature` header.

**Authentication Failures:** Requests with missing or invalid headers, an incorrect signature, or significant timestamp skew will result in a `401 Unauthorized` response.

---

## 1099-DA Cost Basis Sharing Extension

Facilitate accurate IRS Form 1099-DA reporting for your users by enabling seamless sharing of relevant cost basis information calculated within CTC. This extension provides proactive notifications when data is ready and a robust API for retrieving detailed cost basis information, including tax lot specifics crucial for Safe Harbor compliance.

### How It Works

This extension leverages the secure SSO connection established by the core TaxHub integration. The process involves a one-time setup, followed by a notification-triggered data pull:

1.  **One-Time Setup:** During the initial partnership configuration via a dedicated process provided by CTC, you (the partner) provide CTC with a secure HTTPS **Notification Webhook URL**. CTC provides you with **Partner-Level API Credentials** (API Key & Secret) for accessing the Data Pull API and establishes a shared secret for signing/verifying the notification webhooks. Standard OAuth 2.0 Client ID exchange also occurs for the base SSO flow.
2.  **Update Notification:** When cost basis data relevant to your platform (e.g., for assets transferred to/from your exchange, focusing on data needed for your 1099-DA) is updated or explicitly prepared by the user within CTC, our system sends a lightweight, cryptographically signed notification payload to your registered webhook URL. This signals that new data is available for retrieval without requiring you to poll constantly.
3.  **Data Retrieval:** Upon receiving and successfully verifying the notification, your backend system uses the provided **Partner-Level API Credentials** to make a secure, authenticated `GET` request to the CTC Partner Data API endpoint. This request specifies the `provider_uid` (obtained from the notification or your internal mapping) to retrieve the full, updated, structured cost basis data.
4.  **Data Ingestion:** Your system processes the JSON response from the Partner Data API, ingesting the detailed cost basis and acquisition lot information for use in your 1099-DA generation process or other internal systems.

---

### Notification Webhook Specification

CTC uses webhooks to notify your system when relevant cost basis data is updated for a connected user. This mechanism uses a **Shared Secret** (distinct from the API Secret) for verifying incoming webhook authenticity.

- **Method:** `POST`
- **Endpoint:** Your pre-configured Notification Webhook URL.
- **Headers:**
  - `Content-Type: application/json`
  - `X-CTC-Timestamp`: Current Unix timestamp (seconds since epoch). Validate this on your end to prevent replay attacks (e.g., reject if significantly older than current time).
  - `X-CTC-Signature`: HMAC-SHA256 signature (Hex-encoded) of the payload, used for verification.
  - _(Optional)_ `X-CTC-Event-Id`: A unique ID for this specific webhook event attempt. Useful for debugging and idempotency.
- **Signature Verification (`X-CTC-Signature`):**
  1.  **String to Sign:** Concatenate the timestamp (from `X-CTC-Timestamp`) and the raw request body (JSON string) using a period (`.`) as a separator. Example: `1678886400.{"provider_uid":"user123",...}`.
  2.  **Compute HMAC:** Calculate the HMAC-SHA256 hash of the "String to Sign" using the **Shared Secret** established during partner configuration.
  3.  **Compare:** Hex-encode the computed hash and compare it securely against the value provided in the `X-CTC-Signature` header.
- **Request Body (Payload - Example):** The payload is lightweight, designed only to trigger data retrieval.
  ```json
  {
    "provider_uid": "partner_user_abc789",
    "eventType": "ctc.cost_basis.ready",
    "eventTimestamp": 1678886400
  }
  ```
- **Your Response:**
  - **Success:** Upon successful signature and timestamp validation, immediately return an HTTP `2xx` status code (e.g., `200 OK`, `202 Accepted`). **Process the notification asynchronously** (e.g., queue a task to call the Data API) to avoid blocking the webhook response.
  - **Failure:** Return an appropriate `4xx` (e.g., `401 Unauthorized` for signature mismatch, `400 Bad Request` for malformed payload) or `5xx` error code if validation fails or an immediate server error occurs.
- **CTC Retries:** CTC implements a retry mechanism with exponential backoff for webhook deliveries that fail (non-`2xx` response or network errors). Your endpoint should be idempotent (e.g., using `X-CTC-Event-Id` or tracking `eventTimestamp` per `provider_uid`) to handle potential duplicate deliveries gracefully.

---

### Partner Data API: Get Transaction Cost Basis

This API allows you to retrieve detailed, structured cost basis information for specific transactions associated with a user, filterable by various criteria. This is often used after receiving a webhook notification or based on the partner's own schedule (e.g., for year-end reporting).

**Endpoint:**

- **Method:** `GET`
- **Path:** `/v1/partner/cost_basis/transactions/{provider_uid}`
- **Host:** `https://api.cryptotaxcalculator.io` (Production Host)

**Authentication:**

Requires HMAC-SHA256 signed requests as described in the **Common API Authentication** section above.

**Path Parameters:**

| Parameter      | Type   | Required | Description                                                                   |
| -------------- | ------ | -------- | ----------------------------------------------------------------------------- |
| `provider_uid` | String | Yes      | The unique identifier for the user, established during the SSO/OAuth linkage. |

**Query Parameters:**

| Parameter                 | Type   | Required | Description                                                                                                                                      |
| ------------------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| `provider_transaction_id` | String | No       | Filter results to a specific transaction identified by the partner's unique transaction ID (the ID provided by your system for the transaction). |
| `asset_ticker`            | String | No       | Filter results for a specific asset symbol (e.g., `BTC`, `ETH`). Uppercase format expected.                                                      |
| `start_date`              | String | No       | Filter results for transactions occurring **on or after** this date (inclusive). Format: ISO 8601 `YYYY-MM-DD`.                                  |
| `end_date`                | String | No       | Filter results for transactions occurring **on or before** this date (inclusive). Format: ISO 8601 `YYYY-MM-DD`.                                 |
| `limit`                   | Int    | No       | Maximum number of _top-level transaction records_ containing lots to return (Default: 100, Max: 1000).                                           |
| `offset`                  | Int    | No       | Number of _top-level transaction records_ containing lots to skip for pagination (Default: 0).                                                   |

**Response (200 OK):**

A successful request returns a JSON object containing a list of transaction records (`data`) relevant to the filters, each potentially including the detailed cost basis lots associated with that transaction, along with pagination details.

```json
{
  "data": [
    {
      "asset": "BTC", // Asset ticker for the transaction
      "ctc_transaction_id": "ctc_tx_abc123", // CTC's internal ID for the transaction
      "partner_transaction_id": "partner_tx_xyz789", // The partner's transaction ID, if known/correlated by CTC, else null
      "quantity": "0.5", // Total quantity for this transaction_id event. String for high precision.
      "transaction_date": "2023-05-15T10:30:00Z", // Date of the event (e.g., transfer)
      "transaction_type": "transfer_in", // Type of transaction
      "cost_basis_lots": [
        // Array of CostBasisLot objects representing the breakdown for this transaction
        {
          "acquisition_date": "2022-11-20T08:00:00Z",
          "cost_basis_amount": "5000.00",
          "cost_basis_currency": "USD",
          "quantity": "0.2"
        },
        {
          "acquisition_date": "2023-02-10T12:00:00Z",
          "cost_basis_amount": "10000.00",
          "cost_basis_currency": "USD",
          "quantity": "0.3"
        }
      ],
      "total_cost_basis_amount": "15000.00", // Total cost basis for the quantity (sum of lots). String for high precision.
      "cost_basis_currency": "USD" // Primary currency for the total cost basis amount
    }
    // ... more transaction records matching filters ...
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total_records": 1 // Total number of top-level transaction records matching filters
  }
}
```

**Response Field Notes:**

- **`CostBasisLot` Object Structure:** Each object within the `cost_basis_lots` array represents a specific cost basis layer contributing to the transaction. It includes:
  - `acquisition_date` (String ISO 8601): Original acquisition date of the lot.
  - `cost_basis_amount` (String): Cost basis allocated to this portion of the lot.
  - `cost_basis_currency` (String): Currency of the cost basis.
  - `quantity` (String): Quantity from this specific lot used in the transaction.
- **Precision:** All numeric quantity and amount fields are returned as strings to preserve high precision.

---

## Partner Holdings Snapshot Extension

This extension provides partners with a detailed breakdown of individual cost basis lots associated with their platform for a specific user, representing the state of **holdings** captured at a designated point in time (snapshot date). This facilitates accurate reporting, reconciliation, and potentially aids in fulfilling specific regulatory requirements like IRS Form 1099-DA by providing the necessary lot-level detail as of a specific date.

### Purpose & Value

Understanding the specific cost basis lots associated with assets held on a partner's platform is crucial for accurate tax calculations and reporting. When assets are transferred between platforms or have complex histories, determining the correct basis requires a comprehensive view. This extension leverages the user's reconciled transaction history within CTC to provide a precise snapshot of the cost basis lots relevant _to the partner_ at a specific date (e.g., the beginning of a tax year).

This prevents the partner from needing to reconstruct this potentially complex lot-level detail independently and ensures consistency with the user's overall tax position calculated by CTC.

### Data Provided

This extension returns a detailed list of the individual cost basis lots that are attributed to the partner's platform within CTC's records, as they existed on the specified `snapshot_date`. This includes lots for assets currently held or potentially associated with relevant historical transfers.

Each lot includes details such as:

- **Asset Identification:** Ticker symbol (e.g., `BTC`, `ETH`).
- **Acquisition Date:** The original date the specific lot was acquired.
- **Original Cost Basis Amount & Currency:** The cost basis calculated for that lot at acquisition.
- **Remaining Quantity:** The quantity of that specific lot remaining and associated with the partner platform as of the `snapshot_date`.
- **Calculation Timestamp:** When the snapshot data was computed.

### Accessing the Snapshot (Partner Data API: Get Holdings Snapshot)

Access to this detailed lot data, representing the user's holdings relevant to the partner platform _as of the beginning of the first tax year subject to IRS 1099-DA reporting (January 1st, 2025)_, is provided via a dedicated API endpoint. This pre-calculated snapshot serves as a baseline for subsequent reporting periods.

**Endpoint:**

- **Method:** `GET`
- **Path:** `/v1/partner/cost_basis/holdings/{provider_uid}`
- **Host:** `https://api.cryptotaxcalculator.io` (Production Host)

**Authentication:**

Requires HMAC-SHA256 signed requests as described in the **Common API Authentication** section above.

**Path Parameters:**

| Parameter      | Type   | Required | Description                                                                   |
| -------------- | ------ | -------- | ----------------------------------------------------------------------------- |
| `provider_uid` | String | Yes      | The unique identifier for the user, established during the SSO/OAuth linkage. |

**Query Parameters:**

| Parameter | Type | Required | Description                                                            |
| --------- | ---- | -------- | ---------------------------------------------------------------------- |
| `limit`   | Int  | No       | Maximum number of cost basis lots to return (Default: 100, Max: 1000). |
| `offset`  | Int  | No       | Number of cost basis lots to skip for pagination (Default: 0).         |

**Response (200 OK):**

A successful request returns a JSON object containing the effective date of the snapshot and a paginated list of detailed cost basis lots associated with the partner for the given user as of that date.

```json
{
  "snapshot_effective_date": "2025-01-01", // The fixed date for which this baseline snapshot is generated
  "calculation_timestamp": "2024-02-15T10:00:00Z", // When CTC generated this specific response data
  "cost_basis_lots": [
    // Array of CostBasisLot objects relevant to the partner as of snapshot_effective_date
    {
      "asset": "BTC", // Asset Ticker for this lot
      "acquisition_date": "2022-11-20T08:00:00Z",
      "cost_basis_amount": "5000.00",
      "cost_basis_currency": "USD",
      "quantity": "0.2"
    },
    {
      "asset": "BTC",
      "acquisition_date": "2023-01-05T15:30:00Z",
      "cost_basis_amount": "6000.00",
      "cost_basis_currency": "USD",
      "quantity": "0.3"
    },
    {
      "asset": "ETH",
      "acquisition_date": "2022-12-10T11:00:00Z",
      "cost_basis_amount": "1200.00",
      "cost_basis_currency": "USD",
      "quantity": "1.0"
    }
    // ... potentially many more lots across different assets
  ],
  "pagination": {
    "limit": 100, // The limit used for this request
    "offset": 0, // The offset used for this request
    "total_records": 542 // Total number of lots available for this snapshot/user/partner
  }
}
```

**Response Field Notes:**

- **`CostBasisLot` Object Structure:** Each object within the `cost_basis_lots` array represents a specific cost basis layer relevant to the partner as of the `snapshot_effective_date`. It includes:
  _ `asset` (String): The ticker symbol for the asset of this lot.
  _ `acquisition_date` (String ISO 8601): Original acquisition date of the lot.
  _ `cost_basis_amount` (String): Cost basis allocated to this portion of the lot.
  _ `cost_basis_currency` (String): Currency of the cost basis. \* `quantity` (String): Quantity remaining for this specific lot.
- **Fixed Date:** This endpoint always returns the snapshot as of the specific `snapshot_effective_date` indicated (e.g., "2025-01-01").
- **Partner Relevance:** The specific logic determining which lots are associated with the partner depends on Crypto Tax Calculator's internal tracking of wallets and transfers linked to the partner account.
- **Precision:** All numeric quantity and amount fields are returned as strings to preserve high precision.

---

## General Considerations

These points apply generally to the Partner Data API endpoints described in this document.

- **Error Handling:** Both Partner Data API endpoints utilize standard HTTP status codes for errors (e.g., `400 Bad Request`, `401 Unauthorized`, `403 Forbidden`, `404 Not Found`, `429 Too Many Requests`, `500 Internal Server Error`, `503 Service Unavailable`). Error responses include a JSON body with `error.code` (a machine-readable string indicating the error type, e.g., `INVALID_PARAMETER`, `AUTHENTICATION_FAILED`), `error.message` (a human-readable description), and optional `error.details` (an object containing specific context, like invalid parameter names/values) fields for diagnostics.
- **Rate Limiting:** All Partner Data API endpoints are subject to rate limiting to ensure service stability and fair usage. Exceeding defined limits will result in a `429 Too Many Requests` response. Please check the `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (Unix timestamp) response headers for details on your current limits and status. Contact Crypto Tax Calculator for specific rate limit details applicable to your partnership agreement.
- **API Versioning:** Partner Data APIs use URI path versioning (e.g., `/v1/`). Breaking changes (e.g., removing fields, changing required parameters or authentication methods) will be introduced under a new version path (e.g., `/v2/`). Non-breaking changes (e.g., adding new optional parameters or response fields) may occur within the current version.
- **Data Precision:** All numeric fields representing quantities or currency amounts (e.g., `quantity`, `cost_basis_amount`) are returned as strings to preserve high precision and avoid potential floating-point inaccuracies.

---

### Benefits

- **Enhanced Reporting Accuracy:** Provides the detailed lot-level data often required for precise tax form population (e.g., short-term vs. long-term gains on 1099-DA).
- **Simplified Reconciliation:** Allows partners to reconcile their records against the specific cost basis lots calculated by CTC.
- **Reduced Partner Burden:** Avoids the need for partners to reconstruct complex historical cost basis calculations for assets associated with their platform.
- **User Consistency:** Ensures reporting aligns with the user's comprehensively calculated tax position in CTC.

_Further details on specific error codes, advanced configurations, or SDK availability can be provided upon request._
