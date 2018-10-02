# Vipps Recurring API

This is the API documentation for **Vipps Recurring**

**Status:** 🛠 While we have worked closely with selected partners, and believe that this is
_very_ close to production quality, we are more than happy to receive feedback.

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-recurring-api/issues),
[pull requests](https://github.com/vippsas/vipps-recurring-api/pulls),
or contact us at integration@vipps.no.

Document version: 0.1.0.

# Table of Contents
- [Abstract](#abstract)
- [Terminology](#terminology)
- [Core Functionality](#core-functionality)
	- [Typical flow](#typical-flow)
	- [Agreement states](#agreement-states)
	- [Charge states](#charge-states)
	- [HTTP-responses](#http-responses)
- [Authentication](#authentication-and-authorization)
	- [API access token](#api-access-token)
- [Questions or comments](#questions-or-comments)

## Abstract

Vipps Recurring API is an API that delivers recurring functionality for a merchant, for example but not limited to, a newspaper subscription or a smartphone sale with a corresponding cellphone subscription. This is done by requesting agreement on behalf of the user. The user then is then redirected to the App, where he gives consent to the terms of the agreement. The merchant can then send charges that will be automatically charged on the due date. The API documentation can be viewed [`here`](https://vippsas.github.io/vipps-recurring-api/)

## Terminology

| Term |  Description                                    |
|:-----|:----------------------------------------------- |
| Agreement         | An agreement to a subscription with a set of conditions that a end user agrees to.  |
| Charge         | A single payment within an agreement. |
| NIN         | A national identification number, e.g. SSN in Norway ("fødselsnummer", 11 digits).   |
| MSISDN      | A number uniquely identifying a subscription in a GSM or a UMTS mobile network. Simply put, it is the mapping of the telephone number to the SIM card in a mobile phone. See [MSISDN](https://en.wikipedia.org/wiki/MSISDN). |
| Idempotency | The property of endpoints to be called multiple times without changing the result beyond the initial application. |

# Core functionality

## Typical flow

0. Merchant has retrieved a valid token. See "API access token" paragraph.

1. A user requests a subscription from the merchant.

2. An agreement is sent with [`POST:/api/v1/draftAgreement`](https://vippsas.github.io/vipps-recurring-api/#/draft-agreement-controller/registerUsingPOST). In the  respons is a "agreementResource" field that can be used in step 4.

3. The user approves the agreement.

4. The Merchant retrieves the agreement status. Either by getting all agreements
 [`GET:/api/v1/agreement`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/listUsingGET) or the specific agreement
 [`GET:/api/v1/agreement/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/agreement-controller/getUsingGET) per now we do not send callback when the user accepts.

5. The merchant is now free to post charges to the given charge using [`POST:/api/v1/charge/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/createUsingPOST)

6. Later a merchant might want to check if a user has any failed charges and therefore terminate the subscription. This can be done by getting all charges on the users agreement on  [`GET:/api/v1/charge/{agreementId}`](https://vippsas.github.io/vipps-recurring-api/#/charge-controller/listUsingGET_1)

## Facilitation of user subscriptions through Vipps

*Vipps Recurring* offers the ability to facilitate a subscription based payment. We do not manage the subscription on behalf of the user, the merchant is responsible for checking the validity of their


# Agreement states

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `pending`  | Agreement has been created, but not approved by the user in the app yet. |
| 2 | `active` | The Agreement has been confirmed by the end user in the app and can recieve charges                                      |
| 3 | `stopped`  | Agreement has been stopped by the merchant most likely by the End User contacting the merchant to cancel the agreement

# Charge states

| # | State      | Description                                                                          |
|:--|:-----------|:-------------------------------------------------------------------------------------|
| 1 | `pending`  | Charge has been created. |
| 2 | `due` | The charge will be drawn in 8 days, and can now be viewed by the user in the app                                      |
| 3 | `charged`  | Charge has been completed
| 4 | `failed`  | Charge has failed for some reason. I.E Expired card, insufficient funds, etc.

# HTTP responses

This API returns the following HTTP statuses in the responses:

| HTTP status         | Description                                 |
| ------------------- | ------------------------------------------- |
| `200 OK`            | Request successful                          |
| `201 Created`       | Request successful, resource created        |
| `204 No Content`    | Request successful, but empty result        |
| `400 Bad Request`   | Invalid request, see the error for details  |
| `401 Unauthorized`  | Invalid credentials                         |
| `403 Forbidden`     | Authentication ok, but credentials lacks authorization  |
| `404 Not Found`     | The resource was not found  |
| `409 Conflict`      | Unsuccessful due to conflicting resource   |
| `429 Too Many Requests`  | There is currently a limit of max 20 calls per second\*  |
| `500 Server Error`  | An internal Vipps problem.                  |

All error responses contains an `error` object in the body, with details of the problem.

\*: The limit is cautiously set quite low in the production environment, as we want to
monitor performance closely before increasing the limit.
We count HTTP requests per `client_id` and product (ISP and IPP).
For now, all HTTP requests are counted and rate-limited.
We have previously requested data from integrators about volume, times, etc,
but only received this from one integrator.
If you are able to provide data for your solution, please let us know.

# Authentication and authorization

Vipps has to ensure compliance with governing regulations, including
[GDPR](https://ec.europa.eu/info/law/law-topic/data-protection_en).
This means that we have to make sure that Vipps:

* Does not provide information about client, explicitly or implicitly, to unauthorised entities.
* Does not store information accidentally received, which we are not authorised to see.
* Propagate changes quickly if a user opts out or in to Vipps Recurring.
* Does not store personal information, i.e. invoices, which we do not have the right to see.

## API access token

Shortly summarized, you will have to make the following request
(`client_id`, `client_secret` and `Ocp-Apim-Subscription-Key` placeholders must be replaced with real values):

```http
POST https://apitest.vipps.no/api/access-token/jwt-token HTTP/1.1
Host: apitest.vipps.no
Content-Type: application/json
Ocp-Apim-Subscription-Key: <Ocp-Apim-Subscription-Key>

{
	"client_id":"<client_id>",
	"client_secret":"<client_secret>",
	"resource":"https://testapivipps.no/vippsas/recurring-payment-service"
}

```



The request above will return a response similar to this, with the `access_token`:

```http
HTTP 200 OK
{
  "token_type": "Bearer",
  "expires_in": "86398",
  "ext_expires_in": "0",
  "expires_on": "1495271273",
  "not_before": "1495184574",
  "resource": "00000002-0000-0000-c000-000000000000",
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <continued>"
}
```

Every request to the API, needs to have an `Authorization` header with the generated token.

The header in the request to this API should look like this:

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1Ni <continued>
```

# Questions or comments

Please use GitHub's built-in functionality for
[issues](https://github.com/vippsas/vipps-recurring-api/issues),
[pull requests](https://github.com/vippsas/vipps-recurring-api/pulls),
or contact us at integration@vipps.no.

## External documentation

### Technical details about the API

Swagger/OAS API documentation is available on GitHub: https://vippsas.github.io/vipps-recurring-api/

### Getting access to the Vipps Developer Portal

See
[the getting started guide](https://github.com/vippsas/vipps-developers/blob/master/vipps-developer-portal-getting-started.md)
for the Vipps eCommerce API for general information about the Vipps Developer Portal.
This is where you create keys to the API.