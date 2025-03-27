# Illuminate&trade; Serialization REST API

The illuminate-serialization-api repository contains information and examples
for using the Illuminate&trade; Serialization REST API to create serial data
carriers for the Engage and Engage Premium subscriptions on the
Illuminate Platform for product or product variant digital twins.

Each data carrier (digital watermark or QR code) has a unique identifier that,
when scanned with a Digimarc-enabled device, can return metadata that
identifies the product to the device application. This REST API supports the
ability to create data carriers that identify the product as part of a series.

As an example, a company that makes high-tech earbuds offers an extended
warranty to consumers. The instruction booklet contains a serialized QR code
that opens the warranty registration website. After the buyer completes the
registration form and submits payment for the warranty, the warranty system uses
the serialization API to set values for the `warranty_start` and `warranty_end`
date attributes and sets the `has_warranty` flag to `true` for the serial item
whose QR code was scanned. If the warranty expires, the system updates the
`has_warranty` flag to `false`. If the consumer extends the warranty, the system
instead updates the `warranty_end` date.

> The serialization API can create serialized data carriers for products and
> product variants in accounts with the **Engage** or **Engage Premium**
> subscription. Serials for promotional assets aren't supported.

### Character Set and Length

For each serialized item, the API assigns a unique value called a
`serial`, which is generated automatically based on the `strategy` you choose,
the `length` you set, the `symbols` you select, and the number of serial items
you create for the digital twin.

- The **strategy** for the serial can use a sequential number, a random number, or
  a random alphanumeric string.
- The **length** can be 6-20 characters.
- The **symbols** determine the character set for the alphanumeric string, but it
  must be within the [GS1 Application Identifier 21](https://ref.gs1.org/ai/21)
  character set. Numeric strings don't use the `symbols`.

#### Character Sets

Illuminate has four predefined character sets selectable from the user interface.
Specify the symbols for the desired character set or define a custom character
set using your choice of unique [GS1 Application Identifier 21](https://ref.gs1.org/ai/21) symbols.

| Character set | Description                                        | Symbols                                         |
| ------------- | -------------------------------------------------- | ----------------------------------------------- |
| AN-47         | Unambiguous alphanumeric mixed case without vowels | 23456789BCDFGHJKLMNPQRSTVWXZbcdfghjkmnpqrstvwxz |
| GS1-36w       | S1 web-safe alphanumeric upper case                | 0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ            |
| GS1-64w       | GS1 web-safe alphanumeric mixed case               | 23456789BCDFGHJKLMNPQRSTVWXZbcdfghjkmnpqrstvwxz |
| UAN-27        | Unambiguous alphanumeric upper case without vowels | 23456789BCDFGHJKLMNPQRSTVWXZ                    |

- [Getting Started](#getting-started)
- [Requests](#requests)
- [Examples](#examples)
- [Help](#help)

## Getting Started

An account administrator can use the Illuminate UI to get the account ID and
create a digital twin API key for the account you want to work with. The
API key can have read-only (`SERIAL_READ_ONLY`) or read/write (`SERIAL_READ_WRITE`)
permissions. The POST endpoints require read/write permissions.

Include the API key in the authorization header for every request. See the
[Examples](#examples).

The base API URL is usually:

```
https://api.dmrc.app/rest/
```

but for some endpoints, use:

```
https://api.dmrc.app/api/v1/
```

To begin, [configure the account allocation settings](#configure-account-settings)
or the [digital twin allocation settings](#configure-digital-twin-settings). For
information about these settings, see [Allocation Settings](#allocation-settings).

### Digital Twin ID

Some endpoints take a `digitalTwinId` as input. The product's "Digital Twin ID" 
is shown in the last card presented within the UI's detail page for the digital twin.

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

##### Tip

_The API doesn't include an endpoint to GET the allocation settings, but the UI
does display the current settings when creating a new serialization job; in
the digital twin's detail page, navigate to the Serialization tab, then click
the 'Create serial items' button.  The effective settings for the twin will
be displayed, and will be read-only if any serials have previously been 
generated for that twin.

To set the serial allocation parameters, your API key must have
_Create, read, and update_ (`API_EDITOR`) permissions, and the account must have
an Engage or Engage Premium subscription.

#### Flags

Flags are boolean properties, configured at the account level and available
for each serialized item. Each account can have up to five flags. The flags are
unset (false) by default when the serialized item is created and can be
set (true) when a particular event occurs, such as when a consumer registers a 
product. You can also set a flag to true when creating serialized items.

Flags are created by the account administrator in the Illuminate user interface.
Flags can be set or unset only through the API.

#### Attributes

Attributes are custom key-value pairs, configured at the account
level, that enable you to specify a label and a corresponding value for each
serialized item. Each account can have up to 20 attributes. These attributes are
separate and distinct from the custom attributes and values on the digital twin.
For example, a twin can have the custom attributes `color` and `version`, and
the serials could have attributes `registration_date` and `warranty_start`.

## Requests

This section contains request parameters for the available endpoints.

- [Configure Account Settings](#configure-account-settings)
- [Configure Digital Twin Settings](#configure-digital-twin-settings)
- [Start Serialization Job](#start-serialization-job)
- [Get Serialization Job Status](#get-job-status)
- [Get Serials](#get-serials)
- [Create a Single Data Carrier](#create-a-data-carrier)
- [Get Flag Value for Serial](#get-flag-value-for-serial)
- [Get All Flags for Serial](#get-all-flags-for-serial)
- [Get Attribute Value for Serial](#get-attribute-value-for-serial)
- [Get All Attributes for Serial](#get-all-attributes-for-serial)
- [Get All Flags and Attributes for Serial](#get-all-flags-and-attributes-for-serial)
- [Update Flag Value for Serial](#update-flag-value-for-serial)
- [Update Multiple Flags for Serial](#update-multiple-flags-for-serial)
- [Update Attribute Value for Serial](#update-attribute-value-for-serial)
- [Update Multiple Attribute Values for Serial](#update-attribute-value-for-serial)
- [Update Multiple Flag and Attribute Values for Serial](#update-multiple-flag-and-attribute-values-for-serial)

### Configure Account Settings

Use this endpoint to set or update the serialization allocation settings for the
account. For this endpoint, the base API URL is:

```
https://api.dmrc.app/rest/
```

```
POST /accountSerialAllocationSettings
Authorization: ApiKey $API_KEY
```

**Body Parameters**

| Name         | Type    | Required? | Description                                                                                                                                                              |
| ------------ | ------- | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `accountId`  | String  | Yes       | The ID of the account.                                                                                                                                                   |
| `length`     | Integer | Yes       | The length of the serial value in the range [6-20].                                                                                                                      |
| `strategy`   | String  | Yes       | The type of serialization for the account: [RANDOM_ALPHANUMERIC, RANDOM_NUMERIC, SEQUENTIAL_NUMERIC].                                                                    |
| `symbols`    | String  | \*        | The character set for the alphanumeric string within the [GS1 Application Identifier 21](https://ref.gs1.org/ai/21) set. If used, must have 3 or more unique characters. |
| `attributes` | Object  | No        | Creates or updates [attributes](#attributes) for serial items in the account. If provided, overwrites existing attributes.                                               |

> \* Required for the RANDOM_ALPHANUMERIC strategy.

**attributes**

| Name        | Type    | Required? | Description                                                                                                                               |
| ----------- | ------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| `name`      | String  | Yes       | The name of the attribute to create. Attribute names can be 1–20 characters long and include lowercase letters, numbers, and underscores. |
| `mandatory` | Boolean | Yes       | To require a value when a serial item is created, set to `true`.                                                                          |

See the [Configure Account Settings Example](#configure-account-settings-example).

### Configure Digital Twin Settings

Use this endpoint to set the serialization allocation settings for the digital
twin.

These settings are specific to the digital twin for which you want to make
serial watermarks or QR codes. You can use one set of parameters for digital
twin A, and a different set for digital twin B, but once you set these
parameters, they can't be changed for digital twins with existing serial items.

If you don't set the allocation parameters for the digital twin, the API copies
the values set for the account. For this endpoint, the base API URL is:

```
https://api.dmrc.app/rest/
```

```
POST /digitalTwinSerialAllocationSettings
Authorization: ApiKey $API_KEY
```

**Body Parameters**

| Name            | Type   | Required? | Description                                                                                                                                                                                 |
| --------------- | ------ | --------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `accountId`     | String | Yes       | The ID of the account.                                                                                                                                                                      |
| `digitalTwinId` | String | Yes       | The ID of the digital twin (see [Digital Twin Id](#digital-twin-id)).                                                                                                                       |
| `length`        | Number | Yes       | The length of the serial value in the range [6-20].                                                                                                                                         |
| `strategy`      | String | Yes       | The type of serialization for the account: [RANDOM_ALPHANUMERIC, RANDOM_NUMERIC, SEQUENTIAL_NUMERIC].                                                                                       |
| `symbols`       | String | \*        | The [character set](#character-sets) for the alphanumeric string within the [GS1 Application Identifier 21](https://ref.gs1.org/ai/21) set. If used, must have 3 or more unique characters. |

> \* Required for the RANDOM_ALPHANUMERIC strategy.

See the [Configure Digital Twin Settings Example](#configure-digital-twin-settings-example).

### Start Serialization Job

Starts the process of creating serial digital twins. The serials job uses the
parameters in the account or digital twin allocation settings. By default, the
type(s) of data carriers that are created depend on the default carrier types
set for the account. For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
POST /accounts/{accountId}/jobs/serialization
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name        | Type   | Required? | Description            |
| ----------- | ------ | --------- | ---------------------- |
| `accountId` | String | Yes       | The ID of the account. |

**Body Parameters**

| Name                          | Type    | Required? | Description                                                                                                                        |
| ----------------------------- | ------- | --------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `digitalTwinId`               | String  | Yes       | The ID of the digital twin.                                                                                                        |
| `count`                       | Number  | Yes       | The number of serials to create in this job.                                                                                       |
| `identifier`                  | String  | Yes       | The name of the serial job creation request.                                                                                       |
| `attributes`                  | Object  | No        | The attribute names and values to set on creation. If any attributes are marked as `mandatory` (required on creation), a value must be provided. |
| `flags`                       | Array   | No        | The flags that are set to `true` on creation.                                                                                      |
| `overrideDataCarrierDefaults` | Boolean | No        | Indicates whether to override the default `dataCarrierSettings` configured on the account. Default is `false`.                     |
| `domain`                      | String  | \*        | The fully qualified domain to use when creating the QR code. If omitted, the account's default domain is used.                     |
| `qrCode`                      | Boolean | \*        | Indicates whether serials are created with QR codes.                                                                               |
| `urlFormat`                   | String  | \*        | The URL format to use for QR codes: `DigitalLink` or `ShortUrl`.                                                                   |
| `watermark`                   | Boolean | \*        | Indicates whether serials are created with watermarks.                                                                             |

\* Required if `overrideDataCarrierDefaults` is `true`.

See the [Start Serialization Job Example](#start-serialization-job-example).

**Sample Input**

```json
{
  "digitalTwinId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "count": 4,
  "identifier": "my_job",
  "attributes": {
    "req_attribute1": "Attribute value",
    "opt_attribute1": "Another attribute value"
  }
}
```

**Sample Output**

```json
{
  "id": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "status": "PENDING",
  "identifier": "my_job"
}
```

### Get Job Status

After starting a serialization job, you can query the progress by calling the
job status endpoint. For this endpoint, pass the input parameters in the query.

> The length of time needed to complete a serialization job depends on a number
> of factors. Note the top-level `status` property to determine whether the job
> is `COMPLETED`.

For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/jobs/serialization/{jobIdentifier}
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                  |
| --------------- | ------ | --------- | ---------------------------- |
| `accountId`     | String | Yes       | The ID of the account.       |
| `jobIdentifier` | String | Yes       | The `identifier` of the job. |
| `limit`         | Number | No        | The maximum number of jobs to retrieve (default=1000) |
| `cursor`        | String | No        | The `id` of the last job returned in the prior page of results. |

See the [Get Serialization Job Status Example](#get-serialization-job-status-example).

### Get Job CSV

Once the job is completed, you can get the CSV file of the serials data.

For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/jobs/serialization/{jobIdentifier}/csv
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                  |
| --------------- | ------ | --------- | ---------------------------- |
| `accountId`     | String | Yes       | The ID of the account.       |
| `jobIdentifier` | String | Yes       | The `identifier` of the job. |

### Get Jobs

After starting serialization jobs, you can query the progress by calling the
jobs status endpoint. For this endpoint, pass the input parameters in the query.

For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/jobs/serialization
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name        | Type   | Required? | Description            |
| ----------- | ------ | --------- | ---------------------- |
| `accountId` | String | Yes       | The ID of the account. |

**Body Parameters**

| Name     | Type   | Required? | Description                |
| -------- | ------ | --------- | -------------------------- |
| `cursor` | String | No        | The ID of the job cursor.  |
| `limit`  | Number | No        | The limit of jobs to read. |

### Get Serials

After you've created serials for a digital twin, you can list them
to get information like the serial's ID (`id`), the generated serial number
(`serial`), and flag and attribute values. For this endpoint, the base API URL
is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                                                                                                                                                |
| --------------- | ------ | --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `accountId`     | String | Yes       | The ID of the account.                                                                                                                                     |
| `digitalTwinId` | String | Yes       | The ID of the digital twin to return serials for.                                                                                                          |
| `flags`         | String | No        | To filter on flag values, use the format: `flag1=false,flag2=true`.                                                                                        |
| `limit`         | Number | Yes       | The maximum number of results to return.                                                                                                                   |
| `cursor`        | String | Yes       | The ID of the next serial to return. When `pageInfo.hasNextPage` is true, set the `cursor` to the `pageInfo.nextCursor` value to get the next set of data. |
| `serialId`      | String | No        | The serial ID of the serial item to return.                                                                                                                            |
| `serial`        | String | No        | The `serial` value of the serial item to return.                                                                                                           |

**Sample Request**

```
GET /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials?limit=100&cursor=dddd4444-5fe4-32dc-b10a-987654fedcba
Authorization: ApiKey $API_KEY
```

**Sample Output**

```json
"serials": [
    {
        "id": "dddd4444-5fe4-32dc-b10a-987654fedcba",
        "serial": "100001",
        "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
        "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
        "allocationSpace": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
        "attributes": {},
        "created": "2024-12-11T18:09:06.402Z",
        "createdById": "user1",
        "modified": "2024-12-11T18:09:06.402Z",
        "modifiedById": "smh-cloudrun-sa",
        "flags": {
            "flag1": false
        }
    },
    {
        "id": "dddd4444-5fe4-32dc-b10a-987654fedcbb",
        "serial": "100002",
        "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb2",
        "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
        "allocationSpace": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb2",
        "attributes": {},
        "created": "2024-12-11T18:09:06.402Z",
        "createdById": "smh-cloudrun-sa",
        "modified": "2024-12-11T18:09:06.402Z",
        "modifiedById": "smh-cloudrun-sa",
        "flags": {
            "flag1": false
        }
    },
],
  "pageInfo": {
    "nextCursor": "dddd4444-5fe4-32dc-b10a-987654fedcbc",
    "hasNextPage": true
  }
```

### Create a Data Carrier

This endpoint creates a data carrier for the specified serial. You can create
only carrierTypes that are missing and supported by the account's
subscriptions. For example, if an account has an Engage Premium subscription
but the default data carrier is only QR code, you can use this endpoint to
add a watermark to an existing serial.

> When you begin a serialization job, the API creates all default data carrier
> types currently configured for the account. Use this endpoint to create
> carriers that weren't previously created.

The `useDefaultSettings` property is specific to QR codes.

- When creating a digital watermark, `useDefaultSettings` can be `true` or
  `false` or omitted.
- When creating a QR code:
  - If `useDefaultSettings` is `true`, Illuminate creates the QR code using the
    account's default values.
  - If `useDefaultSettings` is `false`, the `shortUrlDomain` and `urlFormat` are
    required; if either is omitted, no QR code is created.
  - If `useDefaultSettings` is omitted, no QR code is created.

For this endpoint, the base API URL is:

```
https://api.dmrc.app/rest/
```

```
POST /serialDataCarrier
Authorization: ApiKey $API_KEY
```

**Body Parameters**

| Name                 | Type    | Required? | Description                                                                                                       |
| -------------------- | ------- | --------- | ----------------------------------------------------------------------------------------------------------------- |
| `accountId`          | String  | Yes       | The ID of the account.                                                                                            |
| `carrierType`        | String  | Yes       | Data carrier type format: `DIGITAL_WATERMARK` or `QR_CODE`.                                                       |
| `serialId`           | String  | Yes       | The `id` of the serial to receive the data carrier.                                                               |
| `useDefaultSettings` | Boolean | No        | Indicates whether to use the account's default settings to create the QR code. If omitted, no QR code is created. |
| `urlFormat`          | String  | No        | Data carrier link format: `DigitalLink` or `ShortUrl`.                                                            |
| `shortUrlDomain`     | String  | No        | The fully qualified domain to use when creating the QR code.                                                      |

See the [Create a Single Data Carrier Example](#create-a-single-data-carrier-example).

### Get Flag Value For Serial

This endpoint returns the value of the specified [flag](#flags) for a serial. For this
endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}/flags/{flagName}
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                          |
| --------------- | ------ | --------- | ------------------------------------ |
| `accountId`     | String | Yes       | The ID of the account.               |
| `digitalTwinId` | String | Yes       | The ID of the digital twin          |
| `serialId`      | String | Yes       | The ID or serial of the serial item |
| `flagName`      | String | Yes       | The name of the flag to check        |

**Sample Output**

The boolean value of the flag is returned in the response body.

```json
false
```

### Get All Flags for Serial

This endpoint returns the available [flags](#flags) for a serial. For this
endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}/flags
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                           |
| --------------- | ------ | --------- | ------------------------------------- |
| `accountId`     | String | Yes       | The ID of the account.                |
| `digitalTwinId` | String | Yes       | The ID of the digital twin.           |
| `serialId`      | String | Yes       | The ID or serial of the serial item. |

**Sample Output**

```json
{
  "flag1": false,
  "flag2": true
}
```

### Get Attribute Value for Serial

This endpoint gets the value for a serial's specified [attribute](#attributes).
If the attribute has no value, nothing is returned; otherwise, the value is
returned in the response body. For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}/attributes/{attributeName}
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                         |
| --------------- | ------ | --------- | ----------------------------------- |
| `accountId`     | String | Yes       | The ID of the account.              |
| `digitalTwinId` | String | Yes       | The ID of the digital twin.         |
| `serialId`      | String | Yes       | The ID or serial of the serial item. |
| `attributeName` | String | Yes       | The name of the attribute to check. |

The string value of the attribute is returned in the response body. If the
serial has no attributes or an attribute has no value, nothing is returned.

**Sample Output**

```json
new attr1 value
```

### Get All Attributes for Serial

This endpoint returns all the [attributes](#attributes) for a serialized item.
For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
GET /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}/attributes
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                          |
| --------------- | ------ | --------- | ------------------------------------ |
| `accountId`     | String | Yes       | The ID of the account.               |
| `digitalTwinId` | String | Yes       | The ID of the digital twin          |
| `serialId`      | String | Yes       | The ID or serial of the serial item | 

**Sample Output**

If the serial has no attributes or an attribute has no value, nothing is
returned; otherwise, this endpoint returns the value for each attribute:

```json
{ "attr1": "attr1 value", "attr2": "attr2 value" }
```

### Get All Flags and Attributes for Serial

This endpoint gets all [flag](#flags) and [attribute](#attributes) values for a
serial. For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```shell
GET /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                          |
| --------------- | ------ | --------- | ------------------------------------ |
| `accountId`     | String | Yes       | The ID of the account.               |
| `digitalTwinId` | String | Yes       | The ID of the digital twin          |
| `serialId`      | String | Yes       | The ID or serial of the serial item |

**Sample Output**

```json
{
  "flags": {
    "flag1": true,
    "flag2": false
  },
  "attributes": {
    "attr1": "attr1 value"
  }
}
```

### Update Flag Value for Serial

This endpoint lets you set the value for a single [flag](#flags) for a serial.
A 200 HTTP response is returned, with data including both the (new) `value` 
and `previous_value` for the flag identified in the request, allowing the caller
to determine whether the flag's effective value was actually changed.

The Include the desired (new) value in the query.

For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
PUT /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}/flags/{flagName}?value={value}
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type    | Required? | Description                           |
| --------------- | ------- | --------- | ------------------------------------- |
| `accountId`     | String  | Yes       | The ID of the account.                |
| `digitalTwinId` | String  | Yes       | The ID of the digital twin.          |
| `serialId`      | String  | Yes       | The ID or serial of the serial item. |
| `flagName`      | String  | Yes       | The name of the flag to update.       |
| `value`         | Boolean | Yes       | The new value for the flag.           |

**Sample Output**

```json
[
  {
    "name": "flag2",
    "value": true,
    "previous_value": false
  }
]
```

### Update Multiple Flags for Serial

This endpoint lets you set the value for one or more [flags](#flags) for a
serial. For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
POST /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                          |
| --------------- | ------ | --------- | ------------------------------------ |
| `accountId`     | String | Yes       | The ID of the account.               |
| `digitalTwinId` | String | Yes       | The ID of the digital twin          |
| `serialId`      | String | Yes       | The ID or serial of the serial item |

**Body Parameters**

| Name    | Type  | Required? | Description                                       |
| ------- | ----- | --------- | ------------------------------------------------- |
| `flags` | Array | Yes       | An object containing the flag name and new value. |

**Sample Body**

```json
{
  "flags": {
    "flag1": true,
    "flag2": false
  }
}
```

**Sample Output**

```json
[
  {
    "name": "flag1",
    "value": true,
    "previous_value": false
  },
  {
    "name": "flag2",
    "value": false,
    "previous_value": true
  }
]
```

### Update Attribute Value for Serial

This endpoint lets you set the value for a single [attribute](#attributes) for a
serial. 

For this endpoint, the base API URL is:

```
https://api.dmrc.app/api/v1
```

```
PUT /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}/attributes/{attributeName}
Authorization: ApiKey $API_KEY
```

**Query Parameters**

| Name            | Type   | Required? | Description                           |
| --------------- | ------ | --------- | ------------------------------------- |
| `accountId`     | String | Yes       | The ID of the account.                |
| `digitalTwinId` | String | Yes       | The ID of the digital twin.          |
| `serialId`      | String | Yes       | The ID or serial of the serial item. |
| `attributeName` | String | Yes       | The name of the attribute to update.  |

**Body Parameters**

| Name    | Type   | Required? | Description                      |
| ------- | ------ | --------- | -------------------------------- |
| `value` | String | Yes       | The new value for the attribute. |

**Sample Body**

```json
{
  "value": "new attr1 value"
}
```

**Sample Output**

```json
[
  {
    "name": "attr1",
    "value": "new attr1 value"
  }
]
```

### Update Multiple Flag and Attribute Values for Serial

This endpoint lets you set the value for multiple [flags](#flags) and
[attributes](#attributes) for a serial item. For this endpoint, the base API URL
is:

```
https://api.dmrc.app/api/v1
```

```
PUT /accounts/{accountId}/digitalTwins/{digitalTwinId}/serials/{serialId}
```

**Query Parameters**

| Name            | Type   | Required? | Description                          |
| --------------- | ------ | --------- | ------------------------------------ |
| `accountId`     | String | Yes       | The ID of the account.               |
| `digitalTwinId` | String | Yes       | The ID of the digital twin          |
| `serialId`      | String | Yes       | The ID or serial of the serial item |

**Body Parameters**

| Name         | Type  | Required? | Description                                                |
| ------------ | ----- | --------- | ---------------------------------------------------------- |
| `flags`      | array | Yes       | An object specifying the flag names and values to set      |
| `attributes` | array | Yes       | An object specifying the attribute names and values to set |

**Sample Body**

```json
{
  "flags": {
    "flag1": true,
    "flag2": true
  },
  "attributes": {
    "attr1": "attr1 value"
  }
}
```

**Sample Output**

```json
{
  "flags": [
    {
      "name": "flag1",
      "value": true,
      "previous_value": false
    },
    {
      "name": "flag2",
      "value": true,
      "previous_value": false
    }
  ],
  "attributes": [
    {
      "name": "attr1",
      "value": "attr1 value"
    }
  ]
}
```

## Examples

The following examples will help you understand what you can do with the
Illuminate Serialization REST API.

- [Configure Account Settings Example](#configure-account-settings-example)
- [Configure Digital Twin Settings Example](#configure-digital-twin-settings-example)
- [Start Serialization Job Example](#start-serialization-job-example)
- [Get Serialization Job Status Example](#get-serialization-job-status-example)
- [List Serials Example](#list-serials-example)
- [Create a Single Data Carrier Example](#create-a-single-data-carrier-example)

### Configure Account Settings Example

This example sets the allocation parameters for the account, with one custom
attribute that's required when serial items are created. If a digital twin
doesn't have its own allocation parameters, these are copied to the twin at the
start of a serialization job.

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/accountSerialAllocationSettings' \
  -d '{
  "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "length": 8,
  "strategy": "RANDOM_NUMERIC",
  "attributes": [
                    {
                        "name": "attribute1",
                        "mandatory": true
                    }
                ]
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

This example sets the allocation parameters for the digital twin. The AN-47
[character set](#character-sets) was selected.

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/digitalTwinSerialAllocationSettings' \
  -d '{
  "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
  "length": 8,
  "strategy": "RANDOM_ALPHANUMERIC",
  "symbols": "23456789BCDFGHJKLMNPQRSTVWXZbcdfghjkmnpqrstvwxz"
}'
```

#### Example Response

```json
{
  "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
  "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
  "serialAllocationSettings": {
    "length": 8,
    "symbols": "23456789BCDFGHJKLMNPQRSTVWXZbcdfghjkmnpqrstvwxz",
    "strategy": "RANDOM_ALPHANUMERIC"
  }
}
```

### Start Serialization Job Example

This example starts a serialization job for a digital twin, creating 10,000 serials:

```shell
curl -H "Authorization: ApiKey $API_KEY" \
  -H 'Content-Type:application/json \
  -X POST '$API_URL/accounts/aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56/jobs/serialization' \
  -d '{
  "domain": "https://example.com/",
  "qrCode": true,
  "urlFormat": "DigitalLink",
  "watermark": true,
  "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
  "identifier": "job_1",
  "attributes": {
    "required_attr1": "attr1 value",
    "optional_attr2": "attr2 value"
  },
  "overrideDataCarrierDefaults": true,
  "count": 10000
}'
```

#### Example Response

```json
{
  "id": "cccc3333-e5f6-1ab2-c34d-fe09dc87ba65",
  "status": "PENDING",
  "identifier": "job_1"
}
```

### Get Serialization Job Status Example

This example gets the job status for the [example job](#start-serialization-job-example), above:

```shell
curl --location 'https://api.dmrc.app/api/v1/accounts/aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56/jobs/serialization/job_1' \
--header 'Authorization: ApiKey $API_KEY' 
```

#### Example Response

This example shows a completed job in which both digital watermarks and QR
codes were created for the serials.

```json
{
  "id": "cccc3333-e5f6-1ab2-c34d-fe09dc87ba65",
  "status": "COMPLETED",
  "identifier": "job_1"
}
```

### List Serials Example

This example shows how to request multiple pages of data, each with 100 serials.

#### First Page

```shell
curl --location 'https://api.dmrc.app/rest/serials?accountId=aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56&digitalTwinId=bbbb2222-ef45-98fe-76dc-ba54fe32dcb1&first=100&order=CREATED_ASC' \
--header 'Authorization: ApiKey $API_KEY'
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

{
    "id": "0193b6e9-0b74-7a14-9931-99eb1cb8547c",
    "serial": "100002",
    "hash": "6B4C45C4ACB6C6B655C5",
    "digitalTwinId": "bbbb2222-ef45-98fe-76dc-ba54fe32dcb1",
    "accountId": "aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56",
    "allocationSpace": "4faf4018-9561-411b-973f-1319b7262fe5",
    "attributes": {},
    "created": "2024-12-11T18:09:06.402Z",
    "createdById": "smh-cloudrun-sa",
    "modified": "2024-12-11T18:09:06.402Z",
    "modifiedById": "smh-cloudrun-sa",
    "flags": {
        "flag1": false,
        "flag2": false
    }
}
```

#### Response Header

| Key             | Value                                |
| --------------- | ------------------------------------ |
| has-next-page   | true                                 |
| next-page-token | eeee5555-5fe4-32dc-b10a-987654fedcba |

#### Second Page

```shell
curl --location 'https://api.dmrc.app/rest/serials?accountId=aaaa1111-e5f6-1ab2-c34d-ab12cd34ef56&digitalTwinId=bbbb2222-ef45-98fe-76dc-ba54fe32dcb1&first=100&order=CREATED_ASC&after=eeee5555-5fe4-32dc-b10a-987654fedcba' \
--header 'Authorization: ApiKey &API_KEY' 
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

**Example Response**

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
    "shortUrlDomain": "https://example.com/"
  }'
```

**Example Response**

The `carrierUrl` is the URL, based on the `shortUrlDomain` that's embedded in
the QR code.

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
