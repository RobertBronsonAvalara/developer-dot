---
layout: post
title: How to Calculate Customs Duties in REST
description: How to use REST to calculate landed costs using AvaTax REST API.
relevantapimethods: ListCrossBorderCodes
date: 2018-03-22 17:00
author: Wayne Myer
comments: true
categories: [avatax, landed cost]
product: blog
doctype: blog
disqus: 1
---

We are excited about our updated customs duty calculation functionality in REST. If you previously used the customs duty calculator in REST, your requests will still work. If you are just starting out or would like to make use of the full functionality of CrossBorder in REST, this post is for you.
<h2>What Are We Shipping?</h2>
For every product you ship across a country border, you first need to know the Harmonized System Code (HS code). While many codes may look alike, the HS code is specific to the destination country and the type of item. You can learn about HS codes and how to look up HS codes here: <a href="/blog/2018/03/13/hs-code-search-in-rest/" target="_blank">HS Code Search in REST</a>. 

In the previous post about HS Code Search, we looked up the HS Code for shipping knitted scarves to Spain. It turns out that knitted scarves shipped to Spain lack a tariff rate. Instead, let's ship a handmade leather handbag. Using the <a href="/api-reference/avatax/rest/v2/methods/Definitions/ListCrossBorderCodes/" target="_blank">HS Code Search API</a>, we know the HS Code for our handbag is 4202210010. This HS code has an <a href="https://en.wikipedia.org/wiki/Ad_valorem_tax" target="_blank" class="italic">ad valorem</a>, or value-based, duty rate associated with it. This means the customs duty is calculated based on the total value of the shipment. Our customs duty calculator also supports unit-based calculations, or number of given measurements of an item, which we will cover in another post.

Here is a simple JSON request to calculate the customs duty and tax on our shipment to Spain:
```json
{
  "type": "SalesInvoice",
  "companyCode": "DEFAULT",
  "isSellerImporterOfRecord" : "true",
  "date": "2018-02-01T00:00:00-07:00",
  "customerCode": "ABC",
  "purchaseOrderNo": "2016-10-13-001",
  "addresses": {
    "ShipTo": {
      "country": "ES"
    },
    "ShipFrom": {
      "city": "Wappingers Falls",
      "region": "NY",
      "country": "US",
      "postalCode": "12590"
    }
  },
  "lines": [
    {
      "number": "1",
      "quantity": 1,
      "amount": 1000,
      "description": "Leather handbag",
      "taxCode": "P0000000",
      "hsCode": "4202210010",
      "parameters": {
      }
    },
    {
      "number": "2",
      "quantity": 1,
      "amount": 50,
      "description": "Shipping",
      "taxCode": "FR010000"
    },
    {
      "number": "3",
      "quantity": 1,
      "amount": 50,
      "description": "Insurance",
      "taxCode": "FR070100"
    }
  ],
  "parameters": {   
  },
  "taxDate": "2018-02-01T00:00:00-07:00",
  "currencyCode": "USD"
}
```


We POST the request to `api/v2/transactions/create` and get this response (abridged):
```json
{
  "id": 78770776,
  "code": "fb060e6e-6c58-4d1d-b1dd-15ffe39d0470",
  ...
  "totalAmount": 1100,
  "totalExempt": 0,
  "totalDiscount": 0,
  "totalTax": 270.93,
  "totalTaxable": 1200,
  "totalTaxCalculated": 270.93,
  ...
  "exchangeRateEffectiveDate": "2018-02-01",
  "exchangeRate": 1,
  "isSellerImporterOfRecord": true,
  ...
  "lines": [
    {
      "id": 39721139,
      "transactionId": 78770776,
      "lineNumber": "1",
      ...
      "lineAmount": 1000,
      ...
      "tax": 249.93,
      "taxableAmount": 1100,
      "taxCalculated": 249.93,
      "taxCode": "P0000000",
      "taxCodeId": 4316,
      ...
      "details": [
        {
          "id": 64751997,
          "transactionLineId": 39721139,
          "transactionId": 78770776,
          "addressId": 46061261,
          "country": "ES",
          "region": "ES",
          "stateFIPS": "ES",
          "exemptAmount": 0,
          "exemptReasonId": 4,
          "inState": false,
          "jurisCode": "ES",
          "jurisName": "SPAIN",
          "jurisdictionId": 200082,
          "stateAssignedNo": "",
          "jurisType": "CNT",
          "jurisdictionType": "Country",
          "nonTaxableAmount": 0,
          "nonTaxableRuleId": 0,
          "nonTaxableType": "RateRule",
          "rate": 0.03,
          "rateRuleId": 5149299,
          "rateSourceId": 4,
          "serCode": "",
          "sourcing": "Destination",
          "tax": 33,
          "taxableAmount": 1100,
          "taxType": "LandedCost",
          "taxName": "ES LandedCost",
          "taxAuthorityTypeId": 45,
          "taxRegionId": 205082,
          "taxCalculated": 33,
          "taxOverride": 0,
          "rateType": "Standard",
          "rateTypeCode": "S",
          "taxableUnits": 1100,
          "nonTaxableUnits": 0,
          "exemptUnits": 0,
          "unitOfBasis": "PerCurrencyUnit",
          "isNonPassThru": false
        },
       ...
  "summary": [
    {
      "country": "ES",
      "region": "ES",
      "jurisType": "Country",
      "jurisCode": "ES",
      "jurisName": "SPAIN",
      "taxAuthorityType": 45,
      "stateAssignedNo": "",
      "taxType": "LandedCost",
      "taxName": "ES LandedCost",
      "rateType": "Standard",
      "taxable": 1100,
      "rate": 0.03,
      "tax": 33,
      "taxCalculated": 33,
      "nonTaxable": 0,
      "exemption": 0
    },
    ...
  ],
  "parameters": {},
  "messages": [
    {
      "summary": "Shipment meets ES import duty de minimis threshold. Therefore, import duty applies.",
      "details": "",
      "refersTo": "LandedCost",
      "severity": "Success",
      "source": "Avalara.AvaTax.Services.Tax"
    }
  ]
}
```


In the lines array, in the first line object you can see the taxCalculated is $33. In the details array for that same line object, the Spain duty for this item has a rate of .03, and since this is an *ad valorem* calculation, the unitOfBasis is “PerCurrencyUnit.” 

<h2>How Does This Thing Work?</h2>
There are four key pieces to trigger a customs duty calculation. 
1. An AvaTaxGlobal subscription is needed to make use of the new calculator. 
2. LandedCost <a href="/api-reference/avatax/rest/v2/models/NexusModel/" target="_blank">nexus</a> is assigned to your company, in the destination country.
3. In the <a href="/api-reference/avatax/rest/v2/models/LineItemModel/" target="_blank">lines object</a>, a valid value in the hsCode field is used to look up the duty rate for the destination country. 
4. The source and destination countries must be different. 

The four parts work together to seamlessly calculate an *ad valorem* duty for your shipment.

<h2>What About DDP/DAP?</h2>
Previously, AvaTax used the terms DAP and DDP to indicate which party was responsible for collecting and remitting the duty and tax. In the new duty calculator, the field <a href="/api-reference/avatax/rest/v2/models/CreateTransactionModel/" target="_blank">isSellerImporterOfRecord</a> determines how AvaTax calculates who pays the customs duty and tax. As the seller, if you are paying or coordinating the customs duty and tax on behalf of your buyer, set this value to true, otherwise use false. You can read more about DAP and DDP here: <a href="/blog/2016/12/15/landed-cost-who-pays/" target="_blank">https://developer.avalara.com/blog/2016/12/15/landed-cost-who-pays/</a>. 

Like other transactions in AvaTax, you can backdate customs duty calculations. However, as of this blog post, January 1, 2018 is the earliest that transactions can be backdated. Using an earlier transaction date will result in an incorrect duty calculation.

Happy global selling!
