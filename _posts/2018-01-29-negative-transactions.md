---
layout: post
title: Negative Transactions
description: How does AvaTax handle transactions that go in different directions?
relevantapimethods: CreateTransaction
date: 2018-01-29 00:00
author: Ted Spence
comments: true
categories: [avatax]
product: blog
doctype: blog
disqus: 1
---

This week I returned a doormat to Costco.  I love Costco's return policies; I was able to buy a mat to try it out, and when we discovered it wasn't a good fit for our garage I was able to return it and buy one that was a better fit.  In the world of sales, refunds are a natural thing - so how do we represent them?

<h3>Defining the Two Parties</h3>

In virtually all tax laws, a transaction involves exactly two parties: a buyer and a seller.  However, the AvaTax API can't just use the words "buyer" and "seller" because we also handle transactions like inventory transfers, purchases, refunds, and so on.  For a full list of transaction types supported by AvaTax, see [DocumentType](/api-reference/avatax/rest/v2/models/enums/DocumentType/).  

AvaTax uses the term "Company" to refer to the business entity that is calling the AvaTax API, and "Customer" to refer to the counterparty for the transaction.  We identify these entities by `companyCode`, which refers to an entity you define with the [CreateCompanies API](/api-reference/avatax/rest/v2/methods/Companies/CreateCompanies/), and `customerCode` which you can define using the [CreateCustomers API](/api-reference/avatax/rest/v2/methods/Customers/CreateCustomers/).

Since AvaTax has a couple of different types of transactions, let's use some examples to illustrate this relationship:

<div class="mobile-table">
    <table class="styled-table">
        <tr>
            <th>Transaction Type</th>
            <th>Example Company</th>
            <th>Example Customer</th>
        </tr>
        <tr>
            <td>SalesOrder or SalesInvoice</td>
            <td>A store selling ice cream</td>
            <td>A kid looking for a treat</td>
        </tr>
        <tr>
            <td>ReturnOrder or ReturnInvoice</td>
            <td>Costco</td>
            <td>Ted, when he returns a doormat that didn't fit in his garage</td>
        </tr>
        <tr>
            <td>InventoryTransferOrder or InventoryTransferInvoice</td>
            <td>One corporate subdivision that has too many computers</td>
            <td>Another corporate subdivision that requested a transfer of computers</td>
        </tr>
        <tr>
            <td>PurchaseOrder or PurchaseInvoice</td>
            <td>A business that sends a request for more copier paper</td>
            <td>The business supplies vendor selling copier paper</td>
        </tr>
    </table>
</div>

<h3>Accounting and Balance Sheets</h3>

With this in mind, the next question is, how do I represent the transaction?  In the real world, we only ever deal with positive amounts of money.  I've never held a negative twenty dollar bill - but I have on occasion owed people money, perhaps if they bought me lunch.

Let's imagine that each time we call the [CreateTransaction API](/api-reference/avatax/rest/v2/methods/Transactions/CreateTransaction/), we are entering a transaction into our bank account.  If the transaction amount is positive, I am adding money to my bank account; if the transaction amount is negative, I am paying money out of my bank account.  We know that most accounting systems are more complex than this, but let's start here anyway!

First thing to remember is that a `committed` transaction is final; whereas a `saved` transaction is still pending.  If your accounting system doesn't think the transaction is final, it should leave the transaction in `saved` status and only call the [CommitTransaction API](/api-reference/avatax/rest/v2/methods/Transactions/CommitTransaction/) when the transaction is complete and verified.

With this conception of a bank account, it's easy to see that anytime a transaction is positive the company's account value increases in their ledger, and when a transaction is negative, the company's account value decreases.  This is why AvaTax uses positive values for the `amount` field when you create a `SalesInvoice`, and negative values for the `amount` field when you create a `ReturnInvoice`.

<h3>Amount vs Quantity</h3>

The next question we need to ask is how we represent the `quantity` field.  Although most sales taxes in the United States are based on the amount of money spent, some taxes are based on the quantity of goods that changed hands - these are often called **excise taxes**.  

An example of an excise tax is a "container deposit" tax, such as [California's CRV tax](https://en.wikipedia.org/wiki/Container_deposit_legislation_in_the_United_States).  These taxes apply a fixed amount for each soda bottle you purchase, regardless of the price of the soda.

Since excise taxes follow a different calculation path than amount-based taxes, Avalara has defined the quantity field as a positive field indicating the absolute quantity of items that changed hands, regardless of direction.  You can see this documentation on the [LineItemModel page](/api-reference/avatax/rest/v2/models/LineItemModel/) in the developer documentation.

<h3>Summing Up</h3>

To finish today's article, let's leave you with a cheat-sheet for the most common use cases:

<div class="mobile-table">
    <table class="styled-table">
        <tr>
            <th>Transaction Type</th>
            <th>Amount Value</th>
            <th>Quantity Value</th>
        </tr>
        <tr>
            <td>SalesOrder or SalesInvoice</td>
            <td>Positive</td>
            <td>Positive</td>
        </tr>
        <tr>
            <td>ReturnOrder or ReturnInvoice</td>
            <td>Negative</td>
            <td>Positive</td>
        </tr>
    </table>
</div>

Here's hoping all your sales are positives!

--Ted Spence
Director, AvaTax API
