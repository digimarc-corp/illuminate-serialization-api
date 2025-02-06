# Illuminate&trade; Serialization REST API

The illuminate-serialization-api repository contains information and examples 
for using the Illuminate&trade; Serialization REST API to create serial data carriers 
on the Illuminate Platform for product or product variant digital twins.

> Serials for promotional assets aren't supported.

Each data carrier (digital watermark or QR code) has a unique identifier that, 
when scanned with a Digimarc-enabled device, can return metadata that 
identifies the product to the device application. This REST API supports the 
ability to create data carriers that identify the product as part of a series.

This is most easily illustrated in the case of a collectible item, such as a
trading card or NFT. The creator might produce a limited-edition set of 100
items, numbered sequentially. Watermarking the items would make them harder
for counterfeiters to copy and easier for legitimate buyers to authenticate.

The Serialization API can create serialized data carriers depending on the
default data carrier configuration and the account's subscription type:

| Subscription | Digital Watermarks | QR Codes |
| ------------ | ------------------ | -------- |
| Automate | Yes | - |
| Engage | - | Yes |
| Engage Premium | Yes | Yes |
| Illuminate | Yes | - |
| Recycle | Yes | - |
| Validate Packaging | Yes | - |


### Character Set and Length

For each serial data carrier, the API assigns a unique value called a
`serial`, which is generated automatically based on the `strategy` you choose,
the `length` you set, the `symbols` you select, and the number of serial data 
carriers you create for the digital twin.

The **strategy** for the serial can use a sequential number or a random number or 
alphanumeric string.  

The **length** can be 6-20 characters.

The **symbols** determine the character set for the alphanumeric string, but it 
must be within the [GS1 Application Identifier 21](https://ref.gs1.org/ai/21)
character set. Numeric strings don't use the `symbols`.

* [Getting Started](#getting-started)
* [Requests](#requests)
* [Examples](#examples)
* [Help](#help)


## Getting Started

An account administrator can use the Illuminate UI to get the account ID and 
create a digital twin API key for the account you want to work with. The 
API key can have read-only (`SERIAL_READ_ONLY`) or read/write (`SERIAL_READ_WRITE`) 
permissions. The POST endpoints require read/write permissions. 

Include the API key in the authorization header for every request. See the 
[Examples](#examples).

The base API URL is:

```
https://api.dmrc.app/rest/
```

To begin, [configure the account allocation settings](#configure-account-settings) 
or the [digital twin allocation settings](#configure-digital-twin-settings). For
information about these settings, see [Allocation Settings](#allocation-settings).

### Digital Twin ID

Some endpoints take a `digitalTwinId` as input. The product's ID is visible in the
Illuminate UI's address bar for a digital twin, but the digital twin's ID isn't. 
An easy way to get the digital twin's `id` is to call `GET /digitalTwins` with 
a filter and include the `productId` in the `product.ids` array:

```json
"filter": {
    "product": {
        "ids": [
            "$PRODUCT_ID"
        ]
    }
}
```

Similarly, to get the digital twin's `id` for a product variant, the filter might 
look like this:

```json
"filter": {
    "productVariant": {
        "ids": [
            "$VARIANT_ID"
        ]
    }
}
```

For more information, see [About the Digital Twin ID](https://github.com/digimarc-corp/illuminate-rest-api#about-the-digital-twin-id) 
and [List Digital Twins](https://github.com/digimarc-corp/illuminate-rest-api#list-digital-twins) 
in the Digital Twins REST API reference.


### Allocation Settings

Before you begin generating serials, you must set the allocation parameters. 
You can set them for the account or digital twin.

To set parameters for each digital twin, see [Configure the Digital Twin Settings](#configure-digital-twin-settings).

To set parameters to use as defaults for twins in the account, see [Configure the Account Settings](#configure-account-settings).

- If the digital twin allocation parameters differ from those on the account, the
digital twin parameters are used. 
- If the account has allocation parameters and the digital twin doesn't, the 
account parameters are copied to the digital twin when the serialization job 
begins. 
- If neither the account nor the digital twin serial allocation parameters 
are set, an error is returned. 

> Once the allocation parameters are set, they can't be changed. Attempting to
set them again results in a 200 HTTP response with the text `"Entity with ID serial_allocation_settings would cause the following conflict: Once the serial allocation settings have been set, they cannot be changed."`

##### Tip
_The API doesn't include an endpoint to GET the allocation settings. To see the 
allocation settings later, [get the job status](#get-job-status) for a 
serialization job, which displays the `length`, `symbols`, and `strategy` that were
applied._

To set the serial allocation parameters, your API key must have 
_Create, read, and update_ (`API_EDITOR`) permissions, and the account must have 
at least one of the following subscriptions:

* Automate
* Illuminate
* Recycle
* Engage
* Engage Premium
* Validate Packaging 

## Requests 

This section contains request parameters for the available endpoints.

* [Configure Account Settings](#configure-account-settings)
* [Configure Digital Twin Settings](#configure-digital-twin-settings)
* [Start Serialization Job](#start-serialization-job)
* [Get Serialization Job Status](#get-job-status)
* [List Serials](#list-serials)
* [Create a Single Data Carrier](#create-a-data-carrier)

### Configure Account Settings

Use this endpoint to set the serialization allocation settings for the account.

```
POST /accountSerialAllocationSettings
Authorization: ApiKey $API_KEY
```

**Body Parameters**

| Name | Type | Required? | Description |
|------|------|---------- | ------------|
| `accountId` | String | Yes | ID of the account. |
| `length` | Integer | Yes | The length of the serial value in the range [6-20].  |
| `strategy` | String | Yes | The type of serialization for the account: [RANDOM_ALPHANUMERIC, RANDOM_NUMERIC, SEQUENTIAL_NUMERIC]. |
| `symbols` | String | * | The character set for the alphanumeric string within the [GS1 Application Identifier 21](https://ref.gs1.org/ai/21) set. If used, must have 3 or more unique characters. |

> \* Required for the RANDOM_ALPHANUMERIC strategy.

See the [Configure Account Settings Example](#configure-account-settings-example).

### Configure Digital Twin Settings

Use this endpoint to set the serialization allocation settings for the digital
twin.

These settings are specific to the digital twin for which you want to make 
serial watermarks or QR codes. You can use one set of parameters for digital 
twin A, and a different set for digital twin B, but once you set these 
parameters, they can't be changed.

If you don't set the allocation parameters for the digital twin, the API copies
the values set for the account, and they can't be updated later.

```
POST /digitalTwinSerialAllocationSettings
Authorization: ApiKey $API_KEY
```

**Body Parameters**

| Name | Type | Required? | Description |
|------|------|---------- | ------------|
| `accountId` | String | Yes | ID of the account. |
| `digitalTwinId` | String | Yes | ID of the digital twin (see [Digital Twin Id](#digital-twin-id)). |
| `length` | Number | Yes | The length of the serial value in the range [6-20]. |
| `strategy` | String | Yes | The type of serialization for the account: [RANDOM_ALPHANUMERIC, RANDOM_NUMERIC, SEQUENTIAL_NUMERIC]. |
| `symbols` | String | * | The character set for the alphanumeric string within the [GS1 Application Identifier 21](https://ref.gs1.org/ai/21) set. If used, must have 3 or more unique characters. |

> \* Required for the RANDOM_ALPHANUMERIC strategy.

See the [Configure Digital Twin Settings Example](#configure-digital-twin-settings-example).


### Start Serialization Job

Starts the process of creating serial digital twins. The serials job uses the 
parameters in the account or digital twin allocation settings and returns a 
job `id` that you can use to query the Job Status endpoint. The type(s) of
data carriers that are created depend on the default carrier types set for the
account.

```
POST /jobs/serialGeneration 
Authorization: ApiKey $API_KEY
```

**Body Parameters**

| Name | Type | Required? | Description |
|------|------|---------- | ------------|
| `accountId` | String | Yes | ID of the account. |
| `digitalTwinId` | String | Yes | ID of the digital twin. |
| `serialCount` | Number | Yes | The number of serials to create in this job. |

See the [Start Serialization Job Example](#start-serialization-job-example).

### Get Job Status

After starting a serialization job, you can query the progress by calling the
job status endpoint. For this endpoint, pass the input parameters in the query.

> The length of time needed to complete a serialization job depends on a number
of factors. Note the top-level `status` property to determine whether the job 
is `COMPLETED`.

```
GET /jobs/status
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name | Type | Required? | Description |
|------|------|---------- | ------------|
| `accountId` | String | Yes | ID of the account. |
| `jobId` | String | Yes | The `id` returned from the `jobs/serialGeneration` endpoint. |

See the [Get Serialization Job Status Example](#get-serialization-job-status-example).


### List Serials

After you've created serials for a digital twin, you can list them
to get information like the serial's ID, status, the serial value, and 
more. For this endpoint, pass the parameters as part of the query.

```
GET /serials
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name | Type | Required? | Description |
|------|------|---------- | ------------|
| `accountId` | String | Yes | ID of the account. |
| `first` | Number | Yes | Number of results to return per page. |
| `order` | String | Yes | Sort order for results: `CREATED_ASC`, `CREATED_DESC`, `MODIFIED_ASC`, or `MODIFIED_DESC`. |
| `digitalTwinId` | String | Yes | List of digital twin IDs to return serials for. |
| `after` | String | No | Set this to the last `serialId` to return in the previous page of serials. |

For example, say you have a digital twin with 1000 serials, which you want in 
pages of 100 serials each. You would set `first` to 100 and get serials 1-100. 
For the next page, you set `after` to the `serialId` of the 100th serial. For 
the third page, you set `after` to the `serialId` of the 200th serial, and so 
on.

If `has-next-page` in the response header is `true`, set `after` to the value 
of the `next-page-token`.

See the [List Serials Example](#list-serials-example).


### Create a Data Carrier

This endpoint creates a data carrier for the specified serial. You can create 
only carrierTypes that are missing and supported by the account's 
subscriptions. For example, if an account has an Engage Premium subscription 
but the default data carrier is only QR code, you can use this endpoint to 
add a watermark to an existing serial. 

> When you begin a serialization job, the API creates all default data carrier 
types currently configured for the account.

```
POST /serialDataCarrier
Authorization: ApiKey $API_KEY
```

**Body Parameters**

| Name | Type | Required? | Description |
|------|------|---------- | ------------|
| `accountId` | String | Yes | ID of the account. |
| `carrierType` | String | Yes | Data carrier type format: `DIGITAL_WATERMARK` or `QR_CODE`. |
| `serialId` | String | Yes | The `id` of the serial to receive the data carrier. |
| `useDefaultSettings` | Boolean | yes | Use default settings for generating the data carrier. |
| `urlFormat` | String | * | Data carrier link format: `DigitalLink` or `ShortUrl`. |
| `shortUrlDomain` | String | * | URL of the data carrier. |

> \* When creating a digital watermark, `useDefaultSettings` can be true or 
false. When creating a QR code, if `useDefaultSettings` is set to false, 
`shortUrlDomain` and `urlFormat` must be set. 

See the [Create a Single Data Carrier Example](#create-a-single-data-carrier-example).


## Examples

The following examples will help you understand what you can do with the 
Illuminate Serialization REST API. 

* [Configure Account Settings Example](#configure-account-settings-example)
* [Configure Digital Twin Settings Example](#configure-digital-twin-settings-example)
* [Start Serialization Job Example](#start-serialization-job-example)
* [Get Serialization Job Status Example](#get-serialization-job-status-example)
* [List Serials Example](#list-serials-example)
* [Create a Single Data Carrier Example](#create-a-single-data-carrier-example)

### Configure Account Settings Example

This example sets the allocation parameters for the account. If a digital twin
doesn't have its own allocation parameters, these are copied to the twin at the
start of a serialization job.

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/accountSerialAllocationSettings' \
  -d '{
  "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "length": 8,
  "strategy": "RANDOM_NUMERIC"
}'
```

#### Example Response

```json
{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "serialAllocationSettings": {
        "length": 8,
        "strategy": "RANDOM_NUMERIC"
    }
}
```

### Configure Digital Twin Settings Example

This example sets the allocation parameters for the digital twin.

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/digitalTwinSerialAllocationSettings' \
  -d '{
  "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
  "length": 8,
  "strategy": "RANDOM_ALPHANUMERIC",
  "symbols": "avcds"
}'
```

#### Example Response

```json
{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
    "serialAllocationSettings": {
        "length": 8,
        "symbols": "avcds",
        "strategy": "RANDOM_ALPHANUMERIC"
    }
}
```

### Start Serialization Job Example

This example starts a serialization job for a digital twin, creating 10,000 serials:

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/jobs/serialGeneration' \
  -d '{
  "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
  "serialCount": 10000
}'
```

#### Example Response

```json
{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "data": {
        "length": 8,
        "strategy": "RANDOM_ALPHANUMERIC"
    },
    "id": "cccc3333-e5f6-1ab2-c34d-fe09dc87ba65",
    "status": "PENDING",
    "type": "SERIAL_GENERATION"
}
```

### Get Serialization Job Status Example

This example gets the job status for the [example job](#start-serialization-job-example), above:

```shell
curl --location 'https://api.dmrc.app/rest/jobs/status?accountId=aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56&jobId=cccc3333-e5f6-1ab2-c34d-fe09dc87ba65' \
--header 'Authorization: ApiKey $API_KEY' \
--data ''
```

#### Example Response

This example shows a completed job in which both digital watermarks and QR 
codes were created for the serials.

```json
{
    "breakdown": {
        "jobItems": [
            {
                "count": 1,
                "items": [
                    {
                        "data": {
                            "count": 10000,
                            "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
                            "nextJobItemId": "ab12cd34-5e6f-a12b-3cd4-12ab34cd56ef",
                            "allocatedRange": [
                                1,
                                10000
                            ]
                        },
                        "status": "COMPLETED"
                    }
                ],
                "type": "SERIAL_ALLOCATION_BATCH"
            },
            {
                "count": 1,
                "items": [
                    {
                        "data": {
                            "range": [
                                1,
                                10000
                            ],
                            "allocationSpace": "ab12cd34-5e6f-a12b-3cd4-12ab34cd56ef"
                        },
                        "status": "COMPLETED"
                    }
                ],
                "type": "SERIAL_DATA_CARRIER_BATCH"
            },
            {
                "count": 1,
                "items": [
                    {
                        "data": {
                            "seed": "90a1bc2d34567ef8901a2bc3d4e56fa7",
                            "range": [
                                1,
                                10000
                            ],
                            "allocationSpace": "ab12cd34-5e6f-a12b-3cd4-12ab34cd56ef"
                        },
                        "status": "COMPLETED"
                    }
                ],
                "type": "SERIAL_HASHING_BATCH"
            }
        ],
        "progress": 1
    },
    "data": {
        "range": [
            1,
            10000
        ],
        "length": 8,
        "symbols": "hapc",
        "strategy": "RANDOM_ALPHANUMERIC",
        "qrCodeStep": true,
        "defaultDomain": "https://example.com/",
        "watermarkStep": true,
        "allocationLevel": "DIGITAL_TWIN",
        "defaultDataCarrier": "DigitalLink"
    },
    "jobItemCounts": {
        "total": 3
    },
    "status": "COMPLETED",
    "type": "SERIAL_GENERATION"
}
```

### List Serials Example

This example shows how to request multiple pages of data, each with 100 serials.

#### First Page
```shell
curl --location 'https://api.dmrc.app/rest/serials?accountId=aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56&digitalTwinId=bbbb2222-ef45-98fe-76dc-ba54fe32dcb1&first=100&order=CREATED_ASC' \
--header 'Authorization: ApiKey $API_KEY' \
--data ''
```

#### Example Response

```json
{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "created": "2024-09-25T19:58:24.186Z",
    "createdById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
    "id": "dddd4444-5fe4-32dc-b10a-987654fedcba",
    "modified": "2024-09-25T19:58:24.186Z",
    "modifiedById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "serial": "hhhhhhha",
    "status": "COMPLETED"
}
```

#### Response Header
| Key | Value |
| --- | ----- |
| has-next-page | true |
| next-page-token | eeee5555-5fe4-32dc-b10a-987654fedcba |

#### Second Page
```shell
curl --location 'https://api.dmrc.app/rest/serials?accountId=aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56&digitalTwinId=bbbb2222-ef45-98fe-76dc-ba54fe32dcb1&first=100&order=CREATED_ASC&after=eeee5555-5fe4-32dc-b10a-987654fedcba' \
--header 'Authorization: ApiKey &API_KEY' \
--data ''
```

#### Example Response

```json
{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "created": "2024-09-25T19:58:24.186Z",
    "createdById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
    "id": "dddd4444-5fe4-32dc-b10a-987654fedcba",
    "modified": "2024-09-25T19:58:24.186Z",
    "modifiedById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "serial": "hhhhapaa",
    "status": "COMPLETED"
}
```

### Create a Single Data Carrier Example

If the account didn't include the desired data carrier as a default when the 
serials were created, you can add the missing carrier type.

#### Digital Watermark

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/serialDataCarrier' \
  -d '{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
    "carrierType": "DIGITAL_WATERMARK",
    "serialId": "dddd4444-5fe4-32dc-b10a-987654fedcba",
    "useDefaultSettings": true
  }'
```

#### QR Code

```shell

curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/serialDataCarrier' \
  -d '{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
    "carrierType": "QR_CODE",
    "serialId": "dddd4444-5fe4-32dc-b10a-987654fedcba",
    "useDefaultSettings": false,
    "urlFormat": "ShortUrl",
    "shortUrlDomain": "https://mydomain.com/"
  }'
```

**Example Responses**

For a digital watermark request:

```json
{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "carrierType": "DIGITAL_WATERMARK",
    "carrierUrl": "",
    "created": "2024-10-03T18:16:10.030Z",
    "createdById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "id": "ffff6666-5fe4-32dc-b10a-987654fedcba",
    "modified": "2024-10-03T18:16:10.030Z",
    "modifiedById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "serialId": "dddd4444-5fe4-32dc-b10a-987654fedcba"
}
```

For a QR code request:

```json
{
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "carrierType": "QR_CODE",
    "carrierUrl": "https://example.com/PswMSzkh",
    "created": "2024-10-03T18:04:06.560Z",
    "createdById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "id": "ffff6666-5fe4-32dc-b10a-987654fedcbb",
    "modified": "2024-10-03T18:04:06.560Z",
    "modifiedById": "ab12cd34-1a23-ddee-ab12cd34ef56",
    "serialId": "dddd4444-5fe4-32dc-b10a-987654fedcba",
    "shortId": "PswMSzkh"
}
```

## Help

You can request assistance by creating a support ticket in the Illuminate UI. 
Log into Illuminate and click the Help icon to get started.
