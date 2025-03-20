# Night Crawler - API Documentation

The Night Crawler API facilitates automated management of Microsoft Azure accounts through a RESTful interface. This documentation outlines the available endpoints, authentication requirements, and expected request/response formats. Please ensure you have the necessary credentials before attempting to access the API.

---

## Table of Contents
- [Base URL](#base-url)
- [Authentication](#authentication)
- [Endpoints](#endpoints)
  - [Get All Accounts](#1-get-all-accounts)
  - [Create New Accounts](#2-create-new-accounts)
  - [Requeue Accounts](#3-requeue-accounts)
- [Notes](#notes)

---

## Base URL

```http
http://night-crawler.adsglory.org/api/accounts
```

---

## Authentication

This API requires a Bearer token for authentication. Include the following headers in your requests:

```http
Authorization: Bearer 1|NewBearerTokenExample123456789
Content-Type: application/json
```

---

## Endpoints

### 1. Get All Accounts

**Endpoint:** `GET /api/accounts`

**Description:** Retrieves a list of accounts with optional filters.

#### Query Parameters:
| Parameter     | Type    | Required | Description |
|--------------|---------|----------|-------------|
| `id`         | Integer | No       | Filter by account ID |
| `foreign_id` | Integer | No       | Filter by foreign ID |
| `username`   | String  | No       | Filter by username |
| `provider`   | String  | No       | Filter by account provider |

#### Response:
- **Success (200 OK):**
```json
{
    "success": true,
    "message": "Request successful",
    "data": [
        {
            "id": 1,
            "foreign_id": 1231,
            "username": "admin1@securemail.com",
            "password": "StrongPass12025",
            "metadata": {
                "api_gateway_url": "http://example/com/api/v1",
                "access_token": "example_access_token",
                "refresh_token": "example_refresh_token",
                "company_abbr": "adv"
            },
            "generated_data": {
                "tenant_id": "ca8d3628-b125-40b1-84b1-c26667057251",
                "client_id": "0042ab34-b04f-4dbc-8e85-cb0698a6980a",
                "client_secret": "WtY8Q~T.-TyX1LvKZtsTBldIW9RGesYPtlSFlc7C"
            },
            "latest_error": null,
            "created_at": "2025-03-20T22:29:55.000000Z",
            "updated_at": "2025-03-20T22:29:55.000000Z",
            "provider": {
                "id": 1,
                "name": "Microsoft"
            }
        },
        {
            "id": 2,
            "foreign_id": 1232,
            "username": "admin2@securemail.com",
            "password": "StrongPass22025",
            "metadata": {
                "api_gateway_url": "http://example/com/api/v1",
                "access_token": "example_access_token",
                "refresh_token": "example_refresh_token",
                "company_abbr": "adv"
            },
            "generated_data": {
                "tenant_id": "ca8d3628-b125-40b1-84b1-c26667057251",
                "client_id": "0042ab34-b04f-4dbc-8e85-cb0698a6980a",
                "client_secret": "WtY8Q~T.-TyX1LvKZtsTBldIW9RGesYPtlSFlc7C"
            },
            "latest_error": null,
            "created_at": "2025-03-20T22:29:55.000000Z",
            "updated_at": "2025-03-20T22:29:55.000000Z",
            "provider": {
                "id": 1,
                "name": "Microsoft"
            }
        }
    ]
}
```
- **Failure (404 Not Found):**
  ```json
  {
    "success": false,
    "message": "No accounts found!"
  }
  ```

---

### 2. Create New Accounts

**Endpoint:** `POST /api/accounts`

**Description:** Stores multiple accounts.

#### Request Body (JSON):
```json
{
    "metadata": {
        "api_gateway_url" : "http://example/com/api/v1",
        "access_token" : "example_access_token",
        "refresh_token" : "example_refresh_token",
        "company_abbr" : "adv"
    },
    "accounts": [
        {
            "foreign_id" : 1231,
            "username" : "admin1@securemail.com",
            "password" : "StrongPass12025",
            "provider" : "Microsoft"
        },
        {
            "foreign_id" : 1232,
            "username" : "admin2@securemail.com",
            "password" : "StrongPass22025",
            "provider" : "Microsoft"
        }
    ]
}
```

#### Validation Rules:
| Parameter     | Type    | Required | Description |
|--------------|---------|----------|-------------|
| `metadata`   | Json  | No       | Additional data for processing |
| `accounts`   | Json   | Yes      | List of accounts to be created |
| `foreign_id` | Integer | No       | External identifier |
| `username`   | String  | Yes      | Account username |
| `password`   | String  | Yes      | Account password |
| `provider`   | String  | Yes      | Account provider name |

#### Response:
- **Success (201 Created):**
```json
{
    "success": true,
    "message": "Request successful",
    "data": {
        "total_accounts": 2,
        "total_accounts_created": 2,
        "total_accounts_failed": 0,
        "failed_accounts": []
    }
}
```
- **Failure (422 Unprocessable Entity - Validation Error):**
```json
{
    "success": false,
    "message": "Validation failed!",
    "errors": {
      "accounts": ["The accounts field is required."]
    }
}
  ```

---

### 3. Requeue Accounts

**Endpoint:** `POST /api/accounts/requeue`

**Description:** Moves accounts back into the processing queue by resetting account-generated data and clearing any recorded errors. This ensures the account is reprocessed as if it were newly created.

#### Request Body (JSON):
```json
{
    "id" : 1
    "foreign_id": NULL,
    "username": NULL,
    "provider": NULL
}
```

#### Validation Rules:
At least one of the following parameters is required:
- `id`
- `foreign_id`
- `username`
- `provider`

#### Response:
- **Success (200 OK):**
```json
{
    "success": true,
    "message": "Request successful",
    "data": {
        "total_accounts": 1,
        "total_accounts_requeued": 1,
        "total_accounts_failed": 0,
        "failed_accounts": []
    }
}
```
- **Failure (404 Not Found):**
```json
{
    "success": false,
    "message": "No accounts found!"
}
```

---

## Notes

- The `generated_data` field contains authentication credentials and other relevant details required for account processing. This data is automatically generated by a PowerShell script after an account is created. When a new account is registered, it enters a **processing queue**, where it awaits execution. The time required for `generated_data` to be generated depends on the length of the queue and the current system load.
- If an account processing attempt fails, the system records the reason for failure in the `latest_error` field. This message provides insights into the issue, such as authentication failures or policy restrictions, allowing users to take corrective action.
- If an account processing attempt fails or if `generated_data` needs to be regenerated, the client must use the Requeue Accounts endpoint to reset and reprocess the account. This can be done by searching for the account using `id`, `foreign_id`, or `username`. Alternatively, the client can requeue all accounts associated with a specific `provider`.
The system will handle re-execution, treating the requeued account as if it were newly created, and no further manual action is required beyond calling the requeue endpoint.
- If the specified `provider` does not exist in the system, it will be automatically created and added to the list of recognized providers. This ensures seamless integration without requiring prior manual setup.

---

This API documentation provides a structured way to interact with the **Night Crawler - Microsoft Azure Automation API**. Ensure proper authentication before calling the API.

