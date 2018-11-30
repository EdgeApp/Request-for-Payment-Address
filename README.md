# Request for Payment Address

Copied from "Request for Payment Address" by Tim Horton (tim@airbitz.co)
Modified by Paul Puey (paul@airbitz.co) per 2015-08-31 meeting with John Lindsay & Aaron Voisine
New modifications and change of spec to optimize QR code density by Paul Puey 2018

## Motivation

Request for Payment Address allows for a fluid interaction between payer and payee
applications. The payer application requests a payment address from a payee wallet. The
wallet generates a new payment address and returns control back to the payer application.

## Specification

A Request for Payment Address is a URI which includes a return URI. The request URI is
launched via the payer application and is handled by the payees preferred wallet application.
The wallet then generates a payment address and builds a bitcoin URI scheme (BIP 21). The
wallet then appends the bitcoin URI to the return URI via a POST addrparameter. The request may 
also include a callback url if the requestor would like the application to forward the user to
a callback URL. This is best implemented for buttons that are tapped to launch an app that will 
service the request and then bounce the user back to the originating app/website.

Bitcoin URIs must follow the format described by BIP 21.

### URI Structure to Request Payment Address

`bitcoin-ret://[action]?[params…]`
`bitcoin-ret://reqaddr?post=https://service.com/1234`

To target specific wallets, `bitcoin-ret` can be replaced by wallet specific URI prefixes such as `edge:`, `bread:`, `multibit:`, etc.

To make altcoin specific address requests, the `bitcoin-ret` prefix can be changed to various altcoin names such as `litecoin-ret:`. Should a specific wallet need to be targetted, the [action] parameter can be changed instead.

Example: `edge://reqaddr-litecoin?[params…]`

### Grammar

#### Payer:

request-uri = `bitcoin-ret://reqaddr?` params

`params = param [ "&" params ]`

`param = [ post / src / callback / maxnum / otherparam ]`

`post = "post=" *qchar`

`src = "src=" *qchar`

`category = "category=" *qchar`

`notes = "notes=" *qchar`

`callback = "callback=" callback-uri`

`net = "net=" [main|test]` `main` is assumed if `net=` is omitted and therefore `network=test` should be the only option ever used.

`maxnum = "maxnum=" maximum number of addresses to return to Payer`

`maxnum` is never guaranteed to be returned to Payer but Payee will not exceed `maxnum` of addresses to be returned. If omitted, `maxnum=1` is assumed. If `max-number=0`, wallet should return as many addresses as it can comfortably handle. Wallet should always return at minimum one address regardless of `max-number` value.

#### Payee:

Payee must submit a POST message to the `post` uri with headers

`Content-Type: application/json`

and a body of format

```
  {
    "source": "Name of Wallet Provider",
    "addresses": [
      { "uri": [ Address URI ] },
      { "uri": [ Address URI ] }
    ]
  }
```

Please see BIP 21 for a definition of qchar​, bitcoinparams, and otherparams.

This spec also supports the optional `category=` param which specifies to the wallet a financial category which all incoming funds may be tagged with. ie. "Income:Wages". The `category=` param should have the format `[Category]:[Subcategory]`.

`[Category]` can only be one of `Income`, `Expense`, `Transfer`, or `Exchange`.

`[Subcategory]` can be a near unlimited set of various categories such as `Salary`, `Food & Dining`, `Clothing`, etc.. Category and Subcategory must be separated by a colon `:` and URI encoded.

Wallets are strongly encouraged to require user interaction before sending a response to the requesting server. This will typically be a popup notifying the user that a website has requested a payment address, with `[OK]` and `[CANCEL]` buttons.

### Example 1

#### 1. Payer creates return URLs.

Success: `https://bitwage.co/user/123`

#### 2. Payer issues a request for a payment address via a `bitcoin-ret:` URI encoding the return URIs. Additional params are included with the URI, in this case `src=` and `category=`.

  `bitcoin-ret://reqaddr`
  
    `?post=https%3A%2F%2Fbitwage.co%2Fuser%2F123`
  
    `&src=Bit%20Wage`
  
    `&maxnum=2`
  
    `&category=Income%3ASalary`

#### 3. Payee receives the `bitcoin-ret:` with action of `reqaddr` and generates two payment address: 
  `1k5hxxtkjrzQV3Q8oAVdKzVcKEnE4EeSf`
  `1z4lkhgYD5Gjd24gHHR677flkj49gDdRe`

#### 4. Payee constructs two bitcoin URIs
  `bitcoin:1k5hxxtkjrzQV3Q8oAVdKzVcKEnE4EeSf`
  `bitcoin:1z4lkhgYD5Gjd24gHHR677flkj49gDdRe`

#### 5. Payee send a message to the POST URI including the address URIs in the body. Payee also
`POST http://bitwage.co/?user=123`
```
  {
    "source": "Edge",
    "addresses": [
      { "uri": "bitcoin:1k5hxxtkjrzQV3Q8oAVdKzVcKEnE4EeSf" },
      { "uri": "bitcoin:1z4lkhgYD5Gjd24gHHR677flkj49gDdRe" }
    ]
  }
```

#### 6. Payer Now can handle the return URI and extract the payment address via the body payload

### Example 2

#### 1. Payer creates a return URI
  Success: `foldapp:request?id=42`
  
#### 2. Payer issues a request for a payment address via a `edge-ret:` URI, encoding the return URIs.
  `edge-ret://reqaddr-litecoin`
  
  `?post=foldapp%3Arequest%3Fid%3D42`
  
  `&src=Foldapp`

  `&category=Expense%3ACoffee`

  `&notes=Refund`

#### 3. Payee handles the `edge-ret:` URI and based on the `reqaddr-litecoin` actions, generates a litecoin address
  `MV5rN5EcX1imDS2gEh5jPJXeiW5QN8YrK3`

#### 4. Payee constructs a litecoin URI
  `litecoin:MV5rN5EcX1imDS2gEh5jPJXeiW5QN8YrK3`

#### 5. Payee send a message to the POST URI including the address URIs in the body. Payee also
`POST http://foldapp.com/?user=123`
```
  {
    "source": "Edge",
    "addresses": [
      { "uri": "bitcoin:1k5hxxtkjrzQV3Q8oAVdKzVcKEnE4EeSf" },
      { "uri": "bitcoin:1z4lkhgYD5Gjd24gHHR677flkj49gDdRe" }
    ]
  }
```

5. Payee issues content of `bitcoin-ret` appending a parameter of `address=`
  `foldapp:request?id=42`

  `&x-source=Airbitz`

  `&address=bitcoin%3A1V8h9kUDXtBEtWwcWRqd5s9A5pxjjdTFy%26label%3DNakamoto`

6. Payee Now can now handle the return URI and extract the payment address via the `address=` parameter


--------------------------------------------
--------------------------------------------

# The BIP 70 (v2) approach 

This is for the use case where a user wishes to receive payment by supplying a third-party with a `PaymentRequest` as defined in BIP 70.

Proposed sequence

1. Payer (e.g. BitWage) creates a link on a website following the BIP 72 URI scheme and informs user: bitcoin:?r=https://example.com/users/abc123
2. User clicks on the link which triggers a wallet launch through standard protocol handlers in the browser
3. Wallet issues a GET request against the `r` link
4. Payer responds with a `PaymentRequest` marked as Version 2 and containing an optional BIP 70 extension field which is a `PaymentRequestSpecification` 
5. Wallet decodes the `PaymentRequestSpecification` to extract salient details (amount to be paid, maximum number of outputs, expiry time, signed with particular Bitcoin address, callback URL etc)
6. Wallet constructs a `PaymentRequest` in accordance with the `PaymentRequestSpecification` and POSTs it to the callback URL
7. Payer decodes the `PaymentRequest` and verifies it matches the `PaymentRequestSpecification`
8. Payer optionally creates a Bitcoin transaction to meet payment if occurring immediately
9. Payer issues `Payment` containing the transaction (or zero bytes if not present)
10. Wallet POSTs `PaymentACK` to indicate end of conversation

The above is just a rough guide to start a conversation.



