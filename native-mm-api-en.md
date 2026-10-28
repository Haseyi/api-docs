# Native MM API Documentation

* [Native API](#Native-API)
* [API Specifications](#API-Specifications)
* [MM](#MM)
* [Webhook](#Webhook)   
     * [Test push events](#Test-push-events)
* [Error Codes](#error-codes)



## Native API


Welcome to the Native API documentation. This document is aimed at Native MM business. 


> It is recommended to testing in the test environment before using the production environment.

Dashboard :

* Test environment dashboard (restricted by IP whitelist): https://mm-sandbox.native.financial/
* Production environment dashboard (restricted by IP whitelist): https://mm.native.financial/

API :

* Test environment: https://mmapi-sandbox.native.financial/
* Production environment: https://mmapi.native.financial/

## API Specifications

- API requests use `HMAC` authentication.

- **Pagination** - Query record lists are all divided into pages, Pagination parameters: `page_num` represents the page number, `page_size` represents the size of each page. API `DTO` uniformly returns `total`, `records`.

- **Country** - Two digit country codes, refer to `ISO 3166-1 alpha-2` standards.

- Time management - API requests and responses return a `UNIX` timestamp, unit being **seconds**, in order to avoid issues due to regional time differences.

- Amount management - All API requests and responses are of the `string` data type in order to avoid precision loss.

- All the requests that have a `body` but don't explicitly define a format are of `JSON` type, `Content-Type: application/json`

- The query interval of all query interfaces must be **less than one month**

- API response format standard-

  | Parameter  |  Type  |                               Description                               |
  | :----: | :----: | :---------------------------------------------------------------------: |
  |  code  |  int   |               Error code. `0`: Normal, non-`0`: Abnormal                |
  |  msg   | string | `SUCCESS` indicates success, error code indicates and describes failure |
  | result | object |                                 Result                                  |

### HMAC Authentication

The institution first needs to apply for the API `key` and API `secret` that will be used when accessing the API.

| Term                   | Description                                                                                                                                                                              |
| ---------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| User ID                | User ID is used to indicate the developer account, is used as the user ID                                                                                                                |
| API key and API secret | Multiple API key + API secret maintained under a User ID, API key is linked with an application, multiple applications are allowed, each application can apply for API access privileges |

#### Client side implementation process:

1. Compose the data that needs to be signed, including-
   - UNIX timestamp, unit being `milliseconds`: the `request` time stamp
   - Request method: `HTTP` method
   - Request API key： API Key
   - Complete request path, including the `URL` parameters: request URI
   - If there is a request `body`, the post conversion `string` of the `body` also needs to be added: string representation of the request payload
2. Client side generates the signature using `HMAC_SHA256` based on the data and API secret
3. Set the Authorization header based on the fixed sequence, i.e. the key is `Authorization`, and the value is: Railone:ApiKey:request time stamp:signature (linked using colon)
4. If the server side sets a password when creating the API key and secret, then an Access-Passphrase header needs to be set, i.e., the `key` is `Access-Passphrase`, and the `value` is the password.
5. Client side sends the data, Authorization header, and the Access-Passphrase header (in case there is a fourth step) to the server side, i.e., the final http header sent is as follows:
   - Authorization：Railone:ApiKey:request timestamp:signature
   - Access-Passphrase：Your API Secret passphrase

#### How to build the request body string to be signed:

The parameter names of the request body need to be based on the respective `ASCII` values, pair key and value using `=`, and connect multiple key-value pairs using `&` to form a string.

Here is an example `body`-

```json
{
  "from": "354245345",
  "amount": 190,
  "to": "dfgdfg"
}
```

The payload is converted to-

```text
amount=190&from=354245345&to=dfgdfg
```

#### Code Implementation and Examples:

For illustrative sample code, please refer to [https://github.com/paycrypto-com/paycrypto-sdk-java](https://github.com/paycrypto-com/paycrypto-sdk-java)


## MM


### asset code list

```text
url：/api/v1/mm/customer/asset/list
method：GET
```

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result":  [
        {
            "network": "Ethereum",
            "asset_type": "CRYPTO",
            "asset_code": "USDT",
            "uni_asset_code": "USDT_ERC20",
        {
            "network": "Tron",
            "asset_type": "CRYPTO",
            "asset_code": "USDT",
            "uni_asset_code": "USDT_TRC20",
        },
        {
            "network": "GL BANK Name",
            "asset_type": "FIAT",
            "asset_code": "USD",
            "uni_asset_code": "USD_GLBANK",
        }
  ]
}
```

### Query customer asset 

```text
url：/api/v1/mm/customer/asset
method：GET
```

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "cust_asset_info_list": [
            {
                "network": "Ethereum",
                "asset_type": "CRYPTO",
                "asset_code": "USDT",
                "uni_asset_code": "USDT_ERC20",
                "account_no": "0xa28B0Ac939FC6BaAaDC79a94f425345C60463417",
                "address": "0xa28B0Ac939FC6BaAaDC79a94f425345C60463417",
                "total_balance": 90,
                "available_balance": 90,
                "frozen_balance": 0,
                "status": 1
            },
            {
                "network": "Tron",
                "asset_type": "CRYPTO",
                "asset_code": "USDT",
                "uni_asset_code": "USDT_TRC20",
                "account_no": "TQn2MYFvDxzHDiLRFsjAXbphfux53RGTFA",
                "address": "TQn2MYFvDxzHDiLRFsjAXbphfux53RGTFA",
                "total_balance": 10,
                "available_balance": 10,
                "frozen_balance": 0,
                "status": 1
            },
            {
                "network": "GL BANK Name",
                "asset_type": "FIAT",
                "asset_code": "USD",
                "uni_asset_code": "USD_GLBANK",
                "account_no": "704637748842995188",
                "address": "704637748842995188",
                "total_balance": 10000,
                "available_balance": 10000,
                "frozen_balance": 0,
                "status": 1
            }

        ],
        "sum_asset_list": [
            {
                "asset_code": "USDT",
                "total_balance": 100,
                "available_balance": 100,
                "frozen_balance": 0,
            },
                        {
                "asset_code": "USD",
                "total_balance": 10000,
                "available_balance": 10000,
                "frozen_balance": 0,
            }
        ],
        "total_balance": "10100",
        "total_balance_currency": "USD"
  }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   total_balance   | String |        total_balance              |



### Deposit asset

- Request:

```text
url：/api/v1/mm/customer/asset/deposit
method：POST
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
| uni_asset_code  |  String  |    Required     | uni_asset_code, USDT_TRC20、USDT_ERC20     |
| tx_hash  |  String  |    Required     | tx_hash    |

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": true
}
```

### Query exchange rate

```text
url：/api/v1/mm/customer/asset/exchange-rate
method：GET
```

- Request：

| Parameter |  Type  |   Requirement  | Description   |
| :------------: | :----: | :----------: |:---------- |
| from_asset | String |Optional| from_asset |
| to_asset | String |Optional| to_asset |

- Response：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": [{
        {
            "from_asset": "USDT",
            "to_asset": "USD",
            "exchange_rate": "0.9999"
        },
        {
            "from_asset": "USDC",
            "to_asset": "USD",
            "exchange_rate": "0.9999"
        }
    }]
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |


### exchange asset

- Request:

```text
url: /api/v1/mm/customer/asset/exchange/order
method：POST
```

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
|  from_uni_asset_code  |  String  |    Required     |  USDT_ERC20     |
|  to_uni_asset_code  |  String  |    Required     |  USD     |
|  crypto_amount  |  String  |    Required     |  10000    |

- Response:

```
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
    }
}
```

|    Parameter    |  Type   |      Description                                                     |
| :---------: | :----:   | :--------------------------- |


### exchange asset history

```text
url：/api/v1/mm/customer/asset/exchange/history
method：GET
```

- Request：

| Parameter |  Type  |   Requirement  | Description   |
| :------------: | :----: | :----------: |:---------- |
|  page_num   |  int  |     Optional     |                        Page number                         |
|  page_size  |  int  |     Optional     |                         Page size                          |
| former_time | long  |     Optional     | Time period upper limit, `UNIX` timestamp, Unit: `seconds` |
| latter_time | long  |     Optional     | Time period lower limit, `UNIX` timestamp, Unit: `seconds` |

- Response：

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "page_num": 1,
        "page_size": 10,
        "total": 1,
        "pages": 1,
        "records": [
            {
                "id": 7,
                "tx_id": "2011710781042962432",
                "business_ref_no": "EX_2011710780833247232",
                "account_no": "704637748842995188",
                "address": "704637748842995188",
                "from_uni_asset_code": "USDT_ERC20",
                "to_uni_asset_code": "USD",
                "transaction_type": "sell",
                "amount": "1000",
                "fee": "3",
                "status": "INIT",
                "reason": "",
                "remark": "",
                "created_at": 1768464200000,
                "updated_at": 1768464200000
            }          
        ]
    }

}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |





### Apply for virtual account

Apply for a virtual account. This API only creates a **local application record** with review status `0-PENDING_REVIEW`; the platform ops team then reviews the application, and the virtual account is created at the upstream bank after approval. Use `GET /api/v1/mm/va/list` to poll the review result and the assigned account number.

```text
url：/api/v1/mm/va/apply
method：POST
```

- Request (common fields):

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
| currency  |  String  |    Required     | Virtual account currency, e.g. `USD`     |
| mail  |  String  |    Optional     | Contact email (corporate: please provide a corporate email). Defaults to the registered email of the authenticated customer    |
| client_type  |  String  |    Required     | Account holder type: `retail` / `corporate`     |
| full_name  |  String  |    Optional     | Account holder full name (individual name / registered company name). Auto-composed from `first_name` + `middle_name` + `last_name` when blank    |
| nationality  |  String  |    Required when `client_type=retail`     | Nationality, 3-letter country code    |
| country  |  String  |    Required     | 3-letter country code. `retail`: country of residence; `corporate`: country of incorporation    |
| city  |  String  |    Required     | City    |
| address  |  String  |    Optional     | Full address. Auto-composed from `address_line1` + `address_line2` + `state_province` + `postal_code` when blank    |
| date  |  String  |    Required     | Format `ddMMyyyy`. Date of birth for `retail`, date of incorporation for `corporate`    |
| id_type  |  String  |    Required when `client_type=retail`     | `P`-Passport, `N`-National ID    |
| id_number  |  String  |    Required     | ID number for `retail` / business registration number for `corporate`    |
| corporate_type  |  String  |    Optional     | `FI` / `NON_FI`, only for `corporate`. Defaults to `NON_FI` when blank    |

- Request (extended fields, SGB Client's Required Info 2026-10-13):

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
| first_name  |  String  |    Required     | First name. `retail`: of the individual; `corporate`: of the contact person    |
| middle_name  |  String  |    Optional     | Middle name    |
| last_name  |  String  |    Required     | Last name. `retail`: of the individual; `corporate`: of the contact person    |
| contact_number  |  String  |    Required     | Contact number: country code + phone number, e.g. `+6591234567`    |
| occupation  |  String  |    Required when `client_type=retail`     | Occupation    |
| address_line1  |  String  |    Required     | Residential/registered address line 1, e.g. `123 Main Street`    |
| address_line2  |  String  |    Optional     | Address line 2, e.g. unit number    |
| state_province  |  String  |    Optional     | State / Province    |
| postal_code  |  String  |    Optional     | Postal code (if applicable)    |
| business_nature  |  String  |    Required when `client_type=corporate`     | Business nature    |
| principal_same_as_registered  |  Boolean  |    Optional     | `corporate` only. Principal place of business is same as registered address. Default `false`    |
| principal_address_line1  |  String  |    Required when `principal_same_as_registered=false`     | Principal place of business address line 1    |
| principal_address_line2  |  String  |    Optional     | Principal place of business address line 2    |
| principal_city  |  String  |    Required when `principal_same_as_registered=false`     | Principal place of business city    |
| principal_country  |  String  |    Required when `principal_same_as_registered=false`     | Principal place of business country, 3-letter country code    |
| principal_state_province  |  String  |    Optional     | Principal place of business state/province    |
| principal_postal_code  |  String  |    Optional     | Principal place of business postal code    |
| source_of_funds  |  Array[String]  |    Required     | Source of funds, multi-select. `retail`: `CRYPTOCURRENCY_STAKING` / `CRYPTOCURRENCY_MINING` / `EMPLOYMENT_INCOME` / `INHERITANCE` / `INVESTMENTS_OR_TRADING` / `SALE_OF_ASSETS` / `OTHERS`; `corporate`: `BUSINESS_ACTIVITY` / `BUSINESS_LOAN` / `CRYPTOCURRENCY_MINING` / `CRYPTOCURRENCY_STAKING` / `INVESTMENTS_OR_TRADING` / `SALE_OF_ASSETS` / `SHAREHOLDER_OR_INVESTOR_FUNDS` / `OTHERS`. A document proof (`SOURCE_OF_FUNDS_PROOF`) should be provided for each option declared    |
| source_of_funds_others  |  String  |    Required when `source_of_funds` contains `OTHERS`     | Source of funds - others description    |
| source_of_wealth_same_as_funds  |  Boolean  |    Optional     | Source of wealth is same as source of funds. Default `false`    |
| source_of_wealth  |  Array[String]  |    Required when `source_of_wealth_same_as_funds` is not `true`     | Source of wealth, multi-select (same options as `source_of_funds`). A document proof (`SOURCE_OF_WEALTH_PROOF`) should be provided for each option declared    |
| source_of_wealth_others  |  String  |    Required when `source_of_wealth` contains `OTHERS`     | Source of wealth - others description    |
| tax_declarations  |  Array[Object]  |    Required     | Tax declarations (jurisdictions of tax residence), at least one. Each item: `{"country_of_jurisdiction": "SGP"}` (3-letter country code)    |
| documents  |  Array[Object]  |    Required     | KYC documents, see the table below    |

- `documents` item:

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
| doc_type  |  String  |    Required     | `PROOF_OF_IDENTITY` / `PROOF_OF_RESIDENTIAL_ADDRESS` / `CERTIFICATE_OF_INCORPORATION` / `ANNUAL_RETURNS_FILING` / `CERTIFICATE_OF_INCUMBENCY` / `SOURCE_OF_FUNDS_PROOF` / `SOURCE_OF_WEALTH_PROOF` / `OTHER`    |
| file_name  |  String  |    Optional     | Original file name, e.g. `passport.jpg`    |
| file_base64  |  String  |    Required when `file_url` is blank     | File content as a base64 data string, e.g. `data:image/jpeg;base64,/9j/4AAQ...`. Formats: `jpg` / `jpeg` / `png` / `bmp`, max `4MB` per file. The server uploads it to S3 and stores only the generated file URL    |
| file_url  |  String  |    Required when `file_base64` is blank     | Already-uploaded file URL (used as-is)    |

Document requirements per client type:

|  client_type  |  Required documents  |
| :-----------: | :------------------- |
| `retail`  | `PROOF_OF_IDENTITY` + `PROOF_OF_RESIDENTIAL_ADDRESS`    |
| `corporate`  | `CERTIFICATE_OF_INCORPORATION` + (`ANNUAL_RETURNS_FILING` or `CERTIFICATE_OF_INCUMBENCY`)    |

- Request example (retail):

```json
{
    "currency": "USD",
    "mail": "customer@example.com",
    "client_type": "retail",
    "nationality": "SGP",
    "country": "SGP",
    "city": "Singapore",
    "date": "01011990",
    "id_type": "P",
    "id_number": "E1234567A",
    "first_name": "John",
    "middle_name": "Michael",
    "last_name": "Doe",
    "contact_number": "+6591234567",
    "occupation": "Software Engineer",
    "address_line1": "1 Raffles Place",
    "address_line2": "#20-61",
    "state_province": "Central Region",
    "postal_code": "048616",
    "source_of_funds": ["EMPLOYMENT_INCOME", "INVESTMENTS_OR_TRADING"],
    "source_of_wealth_same_as_funds": true,
    "tax_declarations": [
        {"country_of_jurisdiction": "SGP"}
    ],
    "documents": [
        {
            "doc_type": "PROOF_OF_IDENTITY",
            "file_name": "passport.jpg",
            "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
        },
        {
            "doc_type": "PROOF_OF_RESIDENTIAL_ADDRESS",
            "file_name": "utility_bill.png",
            "file_base64": "data:image/png;base64,iVBORw0KGgo..."
        },
        {
            "doc_type": "SOURCE_OF_FUNDS_PROOF",
            "file_name": "payslip.jpg",
            "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
        }
    ]
}
```

- Request example (corporate):

```json
{
    "currency": "USD",
    "mail": "finance@acme.com",
    "client_type": "corporate",
    "full_name": "ACME Trading Pte. Ltd.",
    "country": "SGP",
    "city": "Singapore",
    "date": "15032015",
    "id_number": "201506789K",
    "corporate_type": "NON_FI",
    "first_name": "Jane",
    "last_name": "Smith",
    "contact_number": "+6562345678",
    "business_nature": "International trade",
    "address_line1": "10 Anson Road",
    "address_line2": "#15-02",
    "state_province": "Central Region",
    "postal_code": "079903",
    "principal_same_as_registered": true,
    "source_of_funds": ["BUSINESS_ACTIVITY"],
    "source_of_wealth_same_as_funds": true,
    "tax_declarations": [
        {"country_of_jurisdiction": "SGP"}
    ],
    "documents": [
        {
            "doc_type": "CERTIFICATE_OF_INCORPORATION",
            "file_name": "coi.jpg",
            "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
        },
        {
            "doc_type": "ANNUAL_RETURNS_FILING",
            "file_name": "nar1.jpg",
            "file_base64": "data:image/jpeg;base64,/9j/4AAQSkZJRg..."
        }
    ]
}
```

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": {
        "id": 12,
        "currency": "USD",
        "mail": "customer@example.com",
        "virtual_acct_no": null,
        "virtual_iban": null,
        "virtual_acct_status": null,
        "created_and_assigned": 0,
        "reason": null,
        "kyc_extra": {
            "first_name": "John",
            "middle_name": "Michael",
            "last_name": "Doe",
            "contact_number": "+6591234567",
            "occupation": "Software Engineer",
            "address_line1": "1 Raffles Place",
            "address_line2": "#20-61",
            "state_province": "Central Region",
            "postal_code": "048616",
            "business_nature": null,
            "principal_same_as_registered": false,
            "principal_address_line1": null,
            "principal_address_line2": null,
            "principal_city": null,
            "principal_country": null,
            "principal_state_province": null,
            "principal_postal_code": null,
            "source_of_funds": ["EMPLOYMENT_INCOME", "INVESTMENTS_OR_TRADING"],
            "source_of_funds_others": null,
            "source_of_wealth_same_as_funds": true,
            "source_of_wealth": null,
            "source_of_wealth_others": null,
            "tax_declarations": [
                {"country_of_jurisdiction": "SGP"}
            ],
            "documents": [
                {
                    "doc_type": "PROOF_OF_IDENTITY",
                    "file_name": "passport.jpg",
                    "file_base64": null,
                    "file_url": "https://xxx.s3.xxx.amazonaws.com/payone_sgbva_123_proof_of_identity_ab1.jpeg"
                }
            ]
        },
        "create_time": 1768464200000,
        "update_time": 1768464200000
    }
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   id   | long |        Local application record id              |
|   currency   | String |        Virtual account currency              |
|   mail   | String |        Contact email used for the application              |
|   virtual_acct_no   | String |        Virtual account number, assigned after approval (`null` before that)              |
|   virtual_iban   | String |        Virtual IBAN, assigned after approval (`null` before that)              |
|   virtual_acct_status   | String |        Virtual account status reported by the bank (available after approval)              |
|   created_and_assigned   | int |        Review status: `0`-PENDING_REVIEW, `1`-APPROVED (account created), `2`-REJECTED, `3`-CREATE_FAILED (can be reviewed again)              |
|   reason   | String |        Reject / failure reason              |
|   kyc_extra   | Object |        Extended KYC info submitted with the application (stored locally, not sent to the bank). `documents[].file_url` is the uploaded S3 file URL; `file_base64` is always `null` here. `null` for applications created before 2026-10-13              |
|   create_time / update_time   | long |        `UNIX` timestamp, unit: `milliseconds`              |

Note: only one pending/approved application is allowed per customer per currency. Applying again for the same currency while one is `0`-PENDING_REVIEW or `1`-APPROVED returns an error; re-apply is allowed after `2`-REJECTED or `3`-CREATE_FAILED.


### Query virtual account list

Query all virtual account applications/accounts of the authenticated customer, including the review status, the extended KYC info (`kyc_extra`, with document file URLs) and the assigned virtual account number after approval. Typically used to poll the review result after `apply`.

```text
url：/api/v1/mm/va/list
method：GET
```

- Request (query parameters, all optional):

|  Parameter  | Type  | Whether Required |                        Description                         |
| :---------: | :---: | :--------------: | :-------------------------------------------------------- |
| mail  |  String  |    Optional     | Filter by the contact email used for the application (exact match)    |
| currency  |  String  |    Optional     | Filter by virtual account currency, e.g. `USD` (case-insensitive)    |

- Response:

```json
{
    "code": 0,
    "msg": "SUCCESS",
    "result": [
        {
            "id": 12,
            "currency": "USD",
            "mail": "customer@example.com",
            "virtual_acct_no": "847201935762",
            "virtual_iban": "SG83BANK0000847201935762",
            "virtual_acct_status": "ACTIVE",
            "created_and_assigned": 1,
            "reason": null,
            "kyc_extra": {
                "first_name": "John",
                "middle_name": "Michael",
                "last_name": "Doe",
                "contact_number": "+6591234567",
                "occupation": "Software Engineer",
                "address_line1": "1 Raffles Place",
                "address_line2": "#20-61",
                "state_province": "Central Region",
                "postal_code": "048616",
                "business_nature": null,
                "principal_same_as_registered": false,
                "principal_address_line1": null,
                "principal_address_line2": null,
                "principal_city": null,
                "principal_country": null,
                "principal_state_province": null,
                "principal_postal_code": null,
                "source_of_funds": ["EMPLOYMENT_INCOME", "INVESTMENTS_OR_TRADING"],
                "source_of_funds_others": null,
                "source_of_wealth_same_as_funds": true,
                "source_of_wealth": null,
                "source_of_wealth_others": null,
                "tax_declarations": [
                    {"country_of_jurisdiction": "SGP"}
                ],
                "documents": [
                    {
                        "doc_type": "PROOF_OF_IDENTITY",
                        "file_name": "passport.jpg",
                        "file_base64": null,
                        "file_url": "https://xxx.s3.xxx.amazonaws.com/payone_sgbva_123_proof_of_identity_ab1.jpeg"
                    }
                ]
            },
            "create_time": 1768464200000,
            "update_time": 1768550600000
        },
        {
            "id": 10,
            "currency": "EUR",
            "mail": "ops@example.com",
            "virtual_acct_no": null,
            "virtual_iban": null,
            "virtual_acct_status": null,
            "created_and_assigned": 2,
            "reason": "KYC address incomplete",
            "kyc_extra": null,
            "create_time": 1768377800000,
            "update_time": 1768464200000
        }
    ]
}
```

| Parameter |  Type  |          Description          |
| :--------: | :----: | :------------------------------ |
|   id   | long |        Local application record id              |
|   currency   | String |        Virtual account currency              |
|   mail   | String |        Contact email used for the application              |
|   virtual_acct_no   | String |        Virtual account number, assigned after approval (`null` before that)              |
|   virtual_iban   | String |        Virtual IBAN, assigned after approval (`null` before that)              |
|   virtual_acct_status   | String |        Virtual account status reported by the bank (available after approval)              |
|   created_and_assigned   | int |        Review status: `0`-PENDING_REVIEW, `1`-APPROVED (account created), `2`-REJECTED, `3`-CREATE_FAILED (can be reviewed again)              |
|   reason   | String |        Reject / failure reason              |
|   kyc_extra   | Object |        Extended KYC info submitted with the application (same structure as the apply response). `null` for applications created before 2026-10-13              |
|   create_time / update_time   | long |        `UNIX` timestamp, unit: `milliseconds`              |



## Webhook


After configure the Webhook in the Dashboard, please verify signature when you received events. The data structure of the event is:

| Parameter| Type|Description |
| --- | --- |--- |
| action |String | Event type  |
| events | String[]|  events |
| events[n].params | Object | Event content |
| events[n].create_time|long |  UTC time |
| events[n].id | String | Event id |

The data structure of header：

| Parameter| Type|Description |
| --- | --- |--- |
| Signature |String | Signature  |
| Timestamp | String | Timestamp |

```
--header Timestamp：1585310160226
--header Signature：UqAwtsx9HF3s5yJh/c8luvUITZNXE/f3aujwndnXLBU=

```
How to verify signature ? [Example](https://github.com/paycrypto-com/paycrypto-sdk-java/blob/master/src/test/java/com/railone/open/api/test/NotificationTest.java)


The data structure of response should as follows. **If you response correct data structure, the event will not be send again**:



| Parameter| Type|Description |
| --- | --- |--- |
| code | int   |  0: Successful, other: Failure |
|msg  |String  | code message |


Response Example:

```

{
   "code": 0,
   "msg":"SUCCESS"
}
```


### Test push events

```text
url：/api/v1/events/test
method：POST
```

- Request:

|  Parameter   |  Type  | Whether Required |        Description        |
| :----------: | :----: | :--------------: | :-----------------------: |

- Response:

| Parameter| Type|Description |
| --- | --- |--- |
| action |String |  deposit-status|
| events[n].params |Object | event parameters |

Example：
```
{
    "action": "test",
    "events": [
        "{\"id\":\"bc76488ddda4\",\"create_time\":1585293811000,\"params\":{}}"
    ]
}

events element convert string to json:
{
       "id": "bc76488ddda4",
       "create_time": 1585293811000,
       "params":{
       }
}
```


## Error Codes


### Business Logic Error Codes

| Status Code | Description                                                 |
| :---------: | ----------------------------------------------------------- |
|      0      | Succesful                                                   |
|   111001    | Request parameter error                                     |
|   130002    | Data not found                                              |


### Identity Authentication Error Codes

| Status Code | Description                 |
| :---------: | --------------------------- |
|   112001    | Request timed out           |
|   112002    | Illegal access privileges   |
|   112003    | Invalid IP address          |
|   112004    | Invalid timestamp           |
|   112005    | Verification failure        |
|   112006    | Invalid verification format |
|   112007    | Invalid signature           |
|   112008    | Specified app key not found |
|   112009    | Invalid app key secret      |
|   112010    | Invalid request header      |

### Abnormal Status Error Codes

| Status Code | Description           |
| :---------: | --------------------- |
|   119001    | Service unusable      |
|   119002    | Communication error   |
|   119003    | Data encryption error |
|   119004    | Data decryption error |
|   119005    | Too many API requests |
|   119006    | Unauthorized API      |
|   119007    | Public key format error |
