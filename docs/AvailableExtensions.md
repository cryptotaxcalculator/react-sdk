# Available Extensions

Extend the core TaxHub functionality with optional modules designed for deeper partner integration.

## 1099-DA Cost Basis Sharing Extension

Facilitate accurate IRS Form 1099-DA reporting for your users by enabling seamless sharing of relevant cost basis information calculated within CTC. This extension provides proactive notifications when data is ready and a robust API for retrieving detailed cost basis information, including tax lot specifics crucial for Safe Harbor compliance.

### How It Works

This extension leverages the secure SSO connection established by the core TaxHub integration. The process involves a one-time setup, followed by a notification-triggered data pull:

1.  **One-Time Setup:** During the initial partnership configuration via a dedicated process provided by CTC, you (the partner) provide CTC with a secure HTTPS **Notification Webhook URL**. CTC provides you with **Partner-Level API Credentials** (API Key & Secret) for accessing the Data Pull API and establishes a shared secret for signing/verifying the notification webhooks. Standard OAuth 2.0 Client ID exchange also occurs for the base SSO flow.
2.  **Update Notification:** When cost basis data relevant to your platform (e.g., for assets transferred to/from your exchange, focusing on data needed for your 1099-DA) is updated or explicitly prepared by the user within CTC, our system sends a lightweight, cryptographically signed notification payload to your registered webhook URL. This signals that new data is available for retrieval without requiring you to poll constantly.
3.  **Data Retrieval:** Upon receiving and successfully verifying the notification, your backend system uses the provided **Partner-Level API Credentials** to make a secure, authenticated `GET` request to the CTC Partner Data API endpoint. This request specifies the `providerUserId` (obtained from the notification or your internal mapping) to retrieve the full, updated, structured cost basis data.
4.  **Data Ingestion:** Your system processes the JSON response from the Partner Data API, ingesting the detailed cost basis and acquisition lot information for use in your 1099-DA generation process or other internal systems.

---

### Notification Webhook Specification

CTC uses webhooks to notify your system when relevant cost basis data is updated for a connected user. Your system must expose a secure HTTPS endpoint to receive these notifications.

- **Method:** `POST`
- **Endpoint:** Your pre-configured Notification Webhook URL.
- **Headers:**
  - `Content-Type: application/json`
  - `X-CTC-Timestamp`: Current Unix timestamp (seconds since epoch). Validate this on your end to prevent replay attacks (e.g., reject if significantly older than current time).
  - `X-CTC-Signature`: HMAC-SHA256 signature (Hex-encoded) of the payload, used for verification.
  - _(Optional)_ `X-CTC-Event-Id`: A unique ID for this specific webhook event attempt. Useful for debugging and idempotency.
- **Signature Verification (`X-CTC-Signature`):**
  1.  **String to Sign:** Concatenate the timestamp (from `X-CTC-Timestamp`) and the raw request body (JSON string) using a period (`.`) as a separator. Example: `1678886400.{"providerUserId":"user123",...}`.
  2.  **Compute HMAC:** Calculate the HMAC-SHA256 hash of the "String to Sign" using the **Shared Secret** established during partner configuration.
  3.  **Compare:** Hex-encode the computed hash and compare it securely against the value provided in the `X-CTC-Signature` header.
- **Request Body (Payload - Example):** The payload is lightweight, designed only to trigger data retrieval.
  ```json
  {
    "providerUserId": "partner_user_abc789",
    "eventType": "ctc.cost_basis.ready",
    "eventTimestamp": 1678886400
  }
  ```
- **Your Response:**
  - **Success:** Upon successful signature and timestamp validation, immediately return an HTTP `2xx` status code (e.g., `200 OK`, `202 Accepted`). **Process the notification asynchronously** (e.g., queue a task to call the Data API) to avoid blocking the webhook response.
  - **Failure:** Return an appropriate `4xx` (e.g., `401 Unauthorized` for signature mismatch, `400 Bad Request` for malformed payload) or `5xx` error code if validation fails or an immediate server error occurs.
- **CTC Retries:** CTC implements a retry mechanism with exponential backoff for webhook deliveries that fail (non-`2xx` response or network errors). Your endpoint should be idempotent (e.g., using `X-CTC-Event-Id` or tracking `eventTimestamp` per `providerUserId`) to handle potential duplicate deliveries gracefully.

---

### Partner Data API Specification

This API allows you to retrieve the detailed, structured cost basis data calculated by CTC for your connected users, triggered by a notification webhook or your own schedule.

#### Authentication: Signed Requests (HMAC-SHA256)

Requests to the API **MUST** be authenticated using a signature generated with the API Secret component of the Partner-Level API Credentials provided by CTC. All requests must use HTTPS.

1.  **Headers Required:**
    - `X-API-Key`: Your unique API Key provided by CTC.
    - `X-Timestamp`: Current Unix timestamp (seconds since epoch). Requests with timestamps differing significantly from the server time may be rejected.
    - `X-Signature`: The HMAC-SHA256 signature (Base64 encoded).
2.  **Signature Generation:**
    - **String to Sign:** Concatenate the following elements using a newline character (`\n`) as a separator:
      - Unix Timestamp (from `X-Timestamp` header)
      - HTTP Method (e.g., `GET`)
      - Request Path (e.g., `/v1/partner/cost_basis`)
      - Query String (alphabetically sorted by parameter name, e.g., `asset_ticker=BTC&limit=100&providerUserId=...`). If no query string, use an empty string.
      - Request Body (typically empty for GET requests). If no body, use an empty string.
    - **Compute HMAC:** Calculate the HMAC-SHA256 hash of the "String to Sign" using your API Secret as the key.
    - **Encode:** Base64 encode the resulting binary hash. This encoded string is the value for the `X-Signature` header.

Authentication failures (invalid key, signature, timestamp) will result in a `401 Unauthorized` response.

#### Endpoint: Get Cost Basis

Retrieves cost basis information for transactions associated with a specific user, focusing on providing details necessary for assets relevant to your platform and 1099-DA reporting.

- **Method:** `GET`
- **Path:** `/v1/partner/cost_basis`
- **Host:** `https://api.cryptotaxcalculator.io` (Production Host)

#### Query Parameters

| Parameter          | Type   | Required | Description                                                                                                                                                           |
| ------------------ | ------ | -------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `providerUserId`   | String | Yes      | The unique identifier for the user, established during the SSO/OAuth linkage.                                                                                         |
| `transaction_type` | String | No       | Filter results by transaction type (e.g., `transfer_in`, `buy`). If omitted, defaults to types relevant for partner cost basis reporting (consult CTC for specifics). |
| `asset_ticker`     | String | No       | Filter results for a specific asset symbol (e.g., `BTC`, `ETH`). Uppercase format expected.                                                                           |
| `start_date`       | String | No       | Filter results for transactions occurring **on or after** this date (inclusive). Format: ISO 8601 `YYYY-MM-DD`.                                                       |
| `end_date`         | String | No       | Filter results for transactions occurring **on or before** this date (inclusive). Format: ISO 8601 `YYYY-MM-DD`.                                                      |
| `limit`            | Int    | No       | Maximum number of _top-level transaction records_ to return (Default: 100, Max: 1000).                                                                                |
| `offset`           | Int    | No       | Number of _top-level transaction records_ to skip for pagination (Default: 0).                                                                                        |

#### Response (200 OK)

A successful request returns a JSON object containing a list of transaction records (`data`) and pagination details. Each transaction record represents a specific event (e.g., a transfer relevant to the partner) and includes detailed acquisition lot information needed for 1099-DA reporting.

```json
{
  "data": [
    {
      "acquisition_details": [
        // Array of acquisition lots contributing to this transaction's cost basis
        {
          "acquisition_date": "2022-11-20T08:00:00Z", // ISO 8601 timestamp of original acquisition
          "cost_basis_amount": "5000.00", // String for high precision
          "cost_basis_currency": "USD", // Cost basis currency for this lot
          "quantity": "0.2" // Quantity acquired in this lot. String for high precision.
        },
        {
          "acquisition_date": "2023-02-10T12:00:00Z",
          "cost_basis_amount": "10000.00",
          "cost_basis_currency": "USD",
          "quantity": "0.3"
        }
        // ... potentially more lots if FIFO/LIFO etc. requires breakdown
      ],
      "asset": "BTC", // Asset ticker
      "calculation_timestamp": "2024-01-10T15:00:00Z", // When this data was last computed by CTC.
      "cost_basis_currency": "USD", // User's primary reporting currency in CTC.
      "ctc_transaction_id": "ctc_tx_abc123", // CTC's internal ID for the transaction
      "partner_transaction_id": "partner_tx_xyz789", // The partner's transaction ID, if known/correlated by CTC, else null
      "quantity": "0.5", // Total quantity for this transaction_id event. String for high precision.
      "total_cost_basis_amount": "15000.00", // Total cost basis for the quantity. String for high precision.
      "transaction_date": "2023-05-15T10:30:00Z", // Date of the event (e.g., transfer)
      "transaction_type": "transfer_in" // Type of transaction
    }
    // ... more transaction records
  ],
  "pagination": {
    "limit": 100,
    "offset": 0,
    "total_records": 1 // Total number of top-level transaction records matching filters
  }
}
```

**Response Field Notes:**

- **Precision:** `quantity`, `total_cost_basis_amount`, `acquisition_details[].quantity`, `acquisition_details[].cost_basis_amount` are returned as strings to preserve high precision required for financial calculations.
- **`cost_basis_currency`:** Represents the user's primary reporting currency setting within CTC (e.g., USD, AUD, CAD). This is not configurable per request.
- **`acquisition_details`:** Provides the critical breakdown of original acquisition dates and cost basis per lot, essential for accurate capital gains calculation (short-term vs. long-term) required for 1099-DA. The sum of `acquisition_details[].quantity` should equal the top-level `quantity`. The sum of `acquisition_details[].cost_basis_amount` should equal `total_cost_basis_amount`.

#### Error Responses

Standard HTTP status codes are used. Error responses include a JSON body with details.

- `400 Bad Request`: Invalid parameters (e.g., missing `providerUserId`, invalid date format).
- `401 Unauthorized`: Missing or invalid API credentials, invalid signature, or significant timestamp skew.
- `403 Forbidden`: Credentials valid, but lack permission for the requested resource or operation (e.g., user hasn't granted consent).
- `404 Not Found`: `providerUserId` not found or no matching data for the given filters.
- `429 Too Many Requests`: Rate limit exceeded.
- `500 Internal Server Error`: Unexpected server error on CTC's side.
- `503 Service Unavailable`: Temporary outage or maintenance.

**Example Error Body:**

```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Invalid date format for start_date. Use YYYY-MM-DD.",
    "details": {
      "parameter": "start_date",
      "value": "15-01-2023"
    }
  }
}
```

#### Rate Limiting

API requests are subject to rate limiting. Exceeding the limit will result in a `429 Too Many Requests` error. Contact CTC for specific rate limit details applicable to your partnership agreement. Response headers may include `X-RateLimit-Limit`, `X-RateLimit-Remaining`, and `X-RateLimit-Reset` (Unix timestamp indicating when the limit resets).

#### API Versioning

The API uses URI path versioning (e.g., `/v1/`). Breaking changes will be introduced under a new version path. Non-breaking changes (e.g., adding new optional parameters or response fields) may occur within the current version.

---

### Partner Implementation Requirements

_(Consolidated from previous version)_

- **SSO Integration:** Ensure the base TaxHub SDK SSO flow is implemented to establish the `providerUserId` link.
- **Webhook Endpoint:** Create, host, and secure a webhook endpoint capable of:
  - Receiving `POST` requests from CTC IPs.
  - Validating `X-CTC-Timestamp` to prevent replay attacks.
  - Verifying `X-CTC-Signature` using HMAC-SHA256 and the shared secret.
  - Responding quickly with a `2xx` status code.
  - Handling notifications asynchronously (queueing for processing).
  - Implementing idempotency logic.
- **API Client:** Implement a client capable of:
  - Generating HMAC-SHA256 signatures for API requests using the API Key/Secret.
  - Making authenticated `GET` calls to the CTC Partner Data API endpoint.
  - Handling JSON responses, including pagination and error codes.
  - Respecting rate limits.

---

Detailed technical specifications beyond this overview, including specific error codes and potential nuances, are available in our full integration guides upon request.

**Benefits:**

- Reduces user burden and potential errors in manual data transfer.
- Provides accurate, detailed cost basis (including tax lot specifics) needed for 1099-DA Safe Harbor compliance.
- Leverages a secure, push-based notification system combined with a robust pull-based API for efficient and reliable data transfer.

Detailed technical specifications for the webhook payload format, signature verification, and the Partner API (including authentication and data schema) are available upon request.
