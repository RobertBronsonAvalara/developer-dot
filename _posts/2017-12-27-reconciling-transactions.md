---
layout: post
title: Reconciling Transactions
relevantapimethods: ListTransactionsByCompany, VerifyTransaction
date: 2017-12-27 16:00
author: Mark Withers
comments: true
categories: [avatax]
product: blog
doctype: blog
disqus: 1
---

<center><img src="/images/reconciling.jpg" height="350" width="75%"/></center>

Your host system integration is successfully recording and committing transactions in AvaTax - but what about your audit requirement to ensure all the data in AvaTax matches the host system exactly?  You don't have to run Excel mashup reports anymore - AvaTax offers a variety of ways to reconcile your transactions and spot discrepancies in an automated fashion.  

<h3>Listing Transactions via the API</h3>

The first place to start is the <a href="/api-reference/avatax/rest/v2/methods/Transactions/ListTransactionsByCompany/">ListTransactionsByCompany</a> API call in the AvaTax REST namespace.  With this API call, you are able to retrieve all the recorded data in AvaTax and compare with posted host system data for a given date range.  Create either a simple cross-check report which identifies mismatches or get more sophisticated with schemes to auto correct and sync data.    

In this post, we're going to keep the use case simple and focus on providing a discrepancy report run from the <b>host</b> system.  In this example, we will match recorded and posted documents in our host system with recorded and committed transactions in AvaTax for the previous month.  We are interested in matching these values; <b>codes</b>, transaction totalAmount and transaction <b>totalTax</b>.  Any mismatch of these three key-value pairs with our host data will be treated as a discrepancy on our report.   We are also going to assume we need to implement Paging.  AvaTax allows 1,000 transactions returned per call; so we're going to assume more than 1,000 transactions get recorded in any reporting period.  


Setting up the ListTransactionsByCompany API parameters:
<ol>
    <li>Specify the AvaTax <b>companyCode</b> in the GET URL</li>
    <li>Our <b>$filter</b> parameter will be set to date between '2017-12-01' and 2017-12-31'  and <b>status</b> = 'Committed' (for details on setting rest filters <a href=/avatax/filtering-in-rest/>https://developer.avalara.com/avatax/filtering-in-rest/</a>)</li>
        <ul><li>Since some customers have very large data sets, the $filter parameter must have date criteria in it.  If you do not provide a date filter, the API will automatically default to looking at the past 30 days worth of transactions.</li></ul>
    <li>Our <b>$top</b> parameter will be set to 1000 which is the maximum number of transactions returned on a single request</li> 
</ol>

Example request using AvaTax Sandbox:
<div>
    <code>
    https://sandbox-rest.avatax.com/api/v2/companies/Z/transactions?$filter=status%20eq%20'Committed'%20and%20date%20between%20'2016-11-01'%20and%20'2016-11-30'&$top=1000
    </code>
</div>

Example response:
```json
  {
  "@recordsetCount": 60933,
  "value": [
    {
      "id": 257691378,
      "code": "16-21",
      "companyId": 439173,
      "date": "2016-11-30",
      "paymentDate": "1900-01-01",
      "status": "Committed",
      "type": "SalesInvoice",
      "batchCode": "",
      "currencyCode": "CAD",
      "customerUsageType": "",
      "entityUseCode": "",
      "customerVendorCode": "TOB",
      "exemptNo": "",
      "reconciled": false,
      "purchaseOrderNo": "2016-11-28-001",
      "salespersonCode": "",
      "taxOverrideType": "None",
      "taxOverrideAmount": 0,
      "taxOverrideReason": "",
      "totalAmount": 204.98,
      "totalExempt": 16.01,
      "totalDiscount": 0,
      "totalTax": 24.57,
      "totalTaxable": 188.97,
      "totalTaxCalculated": 24.57,
      "adjustmentReason": "NotAdjusted",
      "adjustmentDescription": "",
      "locked": false,
      "region": "ON",
      "country": "CA",
      "version": 1,
      "softwareVersion": "16.8.3.2",
      "originAddressId": 658013785,
      "destinationAddressId": 658013786,
      "exchangeRateEffectiveDate": "2016-11-30",
      "exchangeRate": 1,
      "isSellerImporterOfRecord": false,
      "businessIdentificationNo": "",
      "modifiedDate": "2016-12-06T19:38:55.357",
      "modifiedUserId": 248312,
      "taxDate": "2016-11-30T00:00:00",
      "summary": [],
      "parameters": {
        "AvaTax.LandedCost.Incoterms": "DDP",
        "AvaTax.LandedCost.ShippingMode": "air"
      }
    }
  ],
  "@nextLink": "/api/v2/companies/Z/transactions?%24filter=status+eq+%27Committed%27+and+date+between+%272016-11-01%27+and+%272016-11-30%27&%24top=1&%24skip=1000"
}
```

<h3>Handling Errors</h3>
A successful operation will return a Response Code of 200.  You should be able to see at least two name-value pairs in the result, including: 
```json
"@recordsetCount": 60933
"@nextLink": "/api/v2/companies/Z/transactions?%24filter=status+eq+%27Committed%27+and+date+between+%272016-11-01%27+and+%272016-11-30%27&%24top=1&%24skip=1000"
```

The "recordsetCount" value tells us how many transactions match the $filter criteria.  Since each API call can only return a maximum of 1,000 transactions, this will tell you how many times you need to call the API to retrieve everything.  In this example; we will end up calling the API 61 times to get back all 60,933 transactions.

The "nextLink" value gives us the URL for the next page of information.  If you use this value, AvaTax will keep track of the $skip parameter and allow you to just continue fetching to retrieve the next page of transactions.

There are a few types of errors to look for when calling the ListTransactionsByCompany API:
<ul>
    <li>Did you remember to put the companyCode or valid companyCode in the URL?  If not, you will get an error.</li>
    <li>Did you use the proper filter syntax?  If not, you will get an error.</li>
</ul>

Assuming you collect all the recorded and committed transactions for the reporting period; you can now compare the three values of interest discussed above between the host data and the data retrieved from AvaTax to create a discrepancy report.  Note: you can return even more detailed data to compare with your host data down to the line and tax detail using $include parameter.  

<h3>What's Next</h3>
Once your software decides that a transaction is verified, you can take a variety of steps.  The most common step to take is to call <a href="/api-reference/avatax/rest/v2/methods/Transactions/VerifyTransaction/">VerifyTransaction</a> if you have successfully reconciled the transaction with your host system.  The VerifyTransaction API will update the status of your transaction to Posted.  Alternatively, you could call <a href="/api-reference/avatax/rest/v2/methods/Transactions/CommitTransaction/">CommitTransaction</a> to move the transaction directly to Committed, which allows the transaction to be collected and reported on a tax filing by Avalara's Managed Returns Service.  If you spot errors, the <a href="/api-reference/avatax/rest/v2/methods/Transactions/VoidTransaction/">VoidTransaction</a> API or <a href="/api-reference/avatax/rest/v2/methods/Transactions/AdjustTransaction/">AdjustTransaction</a> API can be used to cancel or update transactions and fix discrepancies.

Yes, you can make it easy for our mutual customers to reconcile their data for accurate reporting and peace of mind!