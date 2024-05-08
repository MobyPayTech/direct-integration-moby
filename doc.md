
# Welcome to Moby API Documentation!

## Introduction

Welcome to Moby API Reference. With Moby API Reference, you can integrate with your system to communicate with Moby services to process your customer payments.

## Prerequisites

All direct merchants who would like to integrate with Moby must register for an account and obtain Merchant ID and API Key, for sub-merchant please check your ID with your parent merchant/merchant acquirer/payment gateway.

## API Integration Flow

1. Customer visits your site and selects a product.
2. Customer checks out with your products and chooses to make payment via Moby.
3. Your site creates a bill via our API.
4. Our API returns the Bill Code and a Bill URL.
5. Your site redirects the customer to Bill URL (bill page).
6. The customer makes payment on our bill page.
7. After the payment is completed, our bill page redirects the customer back to your site to the redirect URL you specify in your initial API request.
8. Upon reaching your site, your site makes another request to our API to retrieve the bill status. (Recommended for security reasons)
9. Your site displays the order status of the checked-out products to the customer.

# Environment

## API Base URL


### Production 
- https://api2.mobypay.my/v2


### Sandbox (Moby)
- https://dev.external-api.moby.my/v2

## API Versioning

| Version | Status                      |
| ------- | --------------------------- |
| v1      | phased out                  |
| v2      | Available                   |

## Security

### API Key
An API key is a token that you provide when making API calls. Include the token in a header parameter called x-api-key.

- x-api-key=1234567890abcdef1234567890abcdef

## Dynamic QR (Variable amount) for POS integration
Call createBill API, and get the billCode from the Response body(JSON). Replace the [billCode] in the URL below from the extracted billCode and generate the QR code base on the FULL URL path.

### Sandbox 
- [https://dev.external-web.moby.my/bc?q=[billCode]](https://dev.external-web.moby.my/bc?q=[billCode])

### Production
- [https://www2.mobypay.my/bc?q=[billCode]](https://www2.mobypay.my/bc?q=[billCode])


# Create Billing

**POST /billing/createBill**

A bill serves as an invoice to the customer. This service will create a bill and return the bill code and bill payment URL (`billUrl`) if successful. Redirect the user to the bill URL for the customer to complete the payment.

## Request

**Body**: `application/json`

| Property         | Type     | Description                                                                                       |
|------------------|----------|---------------------------------------------------------------------------------------------------|
| merchantId       | String   | Required. Merchant ID                                                                             |
| description      | String   | Description of the bill                                                                           |
| amount           | Float    | Required. Amount of the bill                                                                      |
| name             | String   | Customer name                                                                                     |
| mobile           | String   | Customer mobile number                                                                            |
| email            | String   | Customer email address                                                                            |
| ref              | String   | Required. Your own order reference number. We do not check for uniqueness.                        |
| redirectUrl      | String   | Your own page. After payment is complete, the customer will be redirected to this URL.           |
| callbackUrl      | String   | The URL Moby will call to update the status once the customer completes payment. Leave this blank if you don't have one. |
| merchantHomeUrl  | String   | Optional. If this parameter exists, Moby will place a button on the payment webpage to enable the customer to redirect back to the Merchant page if the customer doesn't wish to proceed. |
| submerchantId    | String   | Parent Merchant Only. If you are a merchant acquirer, you need to provide your unique sub-merchant ID for your merchant the payment will be paid to. |
| submerchantName  | String   | Conditional. Parent Merchant Only. If you specify the submerchantId, you need to provide the name of your merchant. |
| productTag       | Array    | A JSON array describing the category of products in this bill, e.g., for a mobile phone purchase `["mobile", "iphone"]` or it could even be the full description of the product(s) `["iPhone 12 Red", "iPhone Cover A123"]`. |

### Sample Request

```json
{
  "merchantId": "string",
  "description": "string",
  "amount": 0.0,
  "name": "string",
  "mobile": "string",
  "email": "string",
  "ref": "string",
  "redirectUrl": "string",
  "callbackUrl": "string",
  "submerchantId": "string",
  "submerchantName": "string",
  "productTag": ["string"]
}
```
## Response

**Body**: `application/json`

| Property   | Type    | Description                                             |
|------------|---------|---------------------------------------------------------|
| status     | Integer | Refer to status in ENUM Definitions.                    |
| errorMsg   | String  | Descriptive error message if an error occurs. If there is no error, this will not be sent. |
| billCode   | String  | Unique alphanumeric reference code for the bill.        |
| ref        | String  | Your own order reference number.                        |
| billUrl    | String  | URL to the Moby payment page for this bill.             |

### Sample Response

```json
{
  "status": 0,
  "errorMsg": "string",
  "billCode": "string",
  "ref": "string",
  "billUrl": "string"
}
```

## Redirect

After the customer has completed payment, Moby will redirect the customer to the redirect URL provided in the request.

We are sending some data in the query parameters. However, it is generally unsafe to trust these parameters and should always rely on a callback or a reactive approach by calling the `getBillStatus` method.

### Parameters

- `billCode`: Unique alphanumeric reference code for the bill.
- `billStatus`: Refer to billStatus in ENUM Definitions.
- `ref`: Your own order reference number.

**Example Redirect URL**:
[https://your.redirect.url/?billCode=string&billStatus=integer&ref=string](https://your.redirect.url/?billCode=string&billStatus=integer&ref=string)

## Callback

After the customer has completed payment, Moby will call the callback URL provided in the request.

### Property

| Property        | Type    | Description                                             |
|-----------------|---------|---------------------------------------------------------|
| billStatus      | Integer | Refer to billStatus in ENUM Definitions.                |
| billStatusDesc  | String  | The bill status description.                            |
| billCode        | String  | Unique alphanumeric reference code for the bill.        |
| amount          | Float   | The amount of the bill.                                 |
| ref             | String  | Your own order reference number.                        |
| timestamp       | String  | Bill created datetime format (RFC3339).                 |

### Sample

```json
{
  "billCode": "string",
  "billStatus": 0,
  "billStatusDesc": "string",
  "amount": 0.0,
  "ref": "string",
  "timestamp": "2019-08-24T00:00:00+00:00"
}
```
## Callback Acknowledgement

Upon receiving the callback, the receiving server is required to respond with HTTP STATUS 200 to indicate that the server received the callback.

**Note:** The callback will only be called once.


## Get Billing Status

**POST /billing/getBillStatus**

Returns the status of a bill based on its bill code.

### Request

**Body**: `application/json`

#### Properties

- `merchantId`: String (Merchant ID)
- `billCode`: String (Reference code of the bill)
- `ref`: String (Merchant transaction ID)

### Sample Request

```json
{
  "merchantId": "string",
  "billCode": "string"
}
```
or
```json
{
  "merchantId": "string",
  "ref": "String"
}
```
### Properties

- `billCode`: String (Unique alphanumeric reference code for the bill)
- `billStatus`: Integer (Refer to billStatus in ENUM Definitions)
- `billStatusDesc`: String (The bill status description)
- `amount`: Float (The amount of the bill)
- `billCode`: String (Unique alphanumeric reference code for the bill)
- `ref`: String (Your own order reference number)
- `timestamp`: String (Bill created datetime format - RFC3339)

If the bill is initiated by a Parent Merchant, the following properties will also be returned:

- `additionalInfo`: Object

### Sample Response using `billCode`

```json
{
  "status": 0,
  "errMsg": "string",
  "billStatus": 0,
  "billStatusDesc": "string",
  "amount": 0.0,
  "billCode": "string",
  "ref": "string",
  "timestamp": "string",
  "additionalInfo": {
    "merchantRate": {
      "percentage": 0.0,
      "fixed": 0.0
    }
  }
}
```
### Sample Response

```json
{
  "status": 0,
  "errorMsg": "string",
  "billCode": "string",
  "ref": "string",
  "billUrl": "string"
}
```
  


## Cancel Billing

**POST /billing/cancelBill**

Cancels bill based on bill code. Only applies to unpaid bills. Returns latest status of bill after API call.

### Request

#### Body

```json

{
  "merchantId": "string",
  "billCode": "string"
}

```
or
```json
{
  "merchantId": "string",
  "ref": "String"
}
```
| Property   | Type    | Description                        |
|------------|---------|------------------------------------|
| merchantId | String  | Required. Merchant ID              |
| billCode   | String  | Required. Reference code of the bill |
| ref        | String  | Merchant transaction ID            |

### Response

#### Body

```json

{
  "status": 0,
  "errMsg": "string",
  "billCode": "string",
  "ref": "string",
  "billStatus": 0,
  "billStatusDesc": "string"
}

```

| Property         | Type    | Description                                                      |
|------------------|---------|------------------------------------------------------------------|
| status           | Integer | Refer to `status` in ENUM Definitions.                          |
| errMsg           | String  | Descriptive error message if has error. If there is no error, this will not be sent. |
| billCode         | String  | Unique alphanumeric reference code for the bill.                 |
| ref              | String  | Your own order reference number.                                 |
| billStatus       | Integer | Refer to `billStatus` in ENUM Definitions.                      |
| billStatusDesc   | String  | The bill status description.                                     |

## Refund Billing

**POST /billing/refundBill**

Refund bill based on bill code. Only applies to paid bills. Only full refunds are allowed. Returns latest status of bill after API call.

### Request

#### Body

```json

{
"merchantId": "string",
"billCode": "string",
"callbackUrl": "string"
}

```
or
```json
{
"merchantId": "string",
"ref": "string",
"callbackUrl": "string"
}
```
| Property     | Type    | Description                                                                                          |
|--------------|---------|------------------------------------------------------------------------------------------------------|
| merchantId   | String  | Required. Merchant ID                                                                                |
| billCode     | String  | Reference code of the bill                                                                          |
| callbackUrl  | String  | The URL Moby will call back when refund status changes from requested to completed or rejected. Response in JSON format as below. Leave this blank if you don't have one. |
| ref          | String  | Merchant transaction ID                                                                              |

### Response

#### Body

```json
{
"status": 0,
"errMsg": "string",
"billCode": "string",
"ref": "string",
"billStatus": 0,
"billStatusDesc": "string"
}
```
| Property        | Type    | Description                                                                                          |
|-----------------|---------|------------------------------------------------------------------------------------------------------|
| status          | Integer | Refer to `status` in ENUM Definitions.                                                               |
| errMsg          | String  | Descriptive error message if has error. If there is no error, this will not be sent.                |
| billCode        | String  | Unique alphanumeric reference code for the bill.                                                     |
| ref             | String  | Your own order reference number.                                                                     |
| billStatus      | Integer | Refer to `billStatus` in ENUM Definitions.                                                           |
| billStatusDesc  | String  | The bill status description.                                                                         |

### Callback

After the refund has been processed, Moby will call the callback URL provided in the request.

```json

{
"billCode": "string",
"billStatus": 0,
"billStatusDesc": "string"
}

```
| Property        | Type    | Description                                        |
|-----------------|---------|----------------------------------------------------|
| billCode        | String  | Unique alphanumeric reference code for the bill   |
| billStatus      | Integer | Refer to `billStatus` in ENUM Definitions         |
| billStatusDesc  | String  | The bill status description                        |

### Callback Acknowledgement

Upon receiving callback, the receiving server is required to respond with HTTP STATUS 200 to indicate that the server received the callback.

Note: The callback will only be called once.

## Get All Billings

**POST /billing/getAllBills**

Returned a JSON array of Bills matching the filter criteria

### Request

#### Body

```json

{
"merchantId": "string",
"submerchantId": "string",
"billStatus": 0,
"startDate": "2019-08-24",
"endDate": "string",
"page": 0
}
```
| Property      | Type   | Description           |
|---------------|--------|-----------------------|
| merchantId    | String | Required. Merchant ID |
| submerchantId |        |                       |


## # ENUM Definitions

### billStatus

| billStatus | billStatusDesc     | Description                                                  | Is it Final? |
|------------|--------------------|--------------------------------------------------------------|--------------|
| 0          | Pending            | API request is valid and successful and payment is pending from customer. | No           |
| 1          | Successful         | Payment is successful.                                       | Yes          |
| 2          | Unsuccessful       | Payment is unsuccessful.                                     | Yes          |
| 3          | Refund Pending     | Refund is pending.                                           | No           |
| 4          | Refunded           | Refund is completed.                                         | Yes          |
| 5          | Refund Rejected    | Refund is rejected.                                          | Yes          |
| 9          | Cancelled          | Bill is cancelled.                                           | Yes          |

### status

| status | errMsg                              | Remark                                                               |
|--------|-------------------------------------|----------------------------------------------------------------------|
| 0      | NIL                                 | API request is valid and successful.                                 |
| 101    | Invalid API Key                     | Check your API Key and/or Merchant ID.                               |
| 102    | API Version not supported           | The API version you are using is not supported with the API Key.     |
| 103    | Missing/Malformed Request Payload  | Check your payload and make sure the required data is sent.          |
| 104    | Invalid Date Range                 | Check your date range sent, make sure it is a valid range.           |
| 201    | Bill not found.                    | The bill you are inquiring does not exist or you have no permission to retrieve. |
| 202    | Bill cancellation not permitted.   | This bill can only be cancelled if the bill status is pending.       |
| 203    | Bill refund not permitted.         | This bill cannot be refunded, could be due to the current bill state or that we did not enable your refund capability. |
| 204    | Bill amount cannot be more than RM10,000. | Maximum amount that can be transacted each time is RM10,000.00.      |
| 205    | Bill code is required.             | Check your payload and make sure the required data is sent.          |
| 502    | Barcode is required.               | Check your payload and make sure the required data is sent.          |
| 301    | Submerchant ID exist.              | This happens if you have added a merchant with this Submerchant ID before, but you're sending the wrong name for this merchant. Your Submerchant ID must be unique for each merchant. |
| 999    | Miscellaneous Error                | Any other errors not mentioned above.                                 |
