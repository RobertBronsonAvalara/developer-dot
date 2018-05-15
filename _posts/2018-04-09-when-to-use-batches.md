---
layout: post
title: When to use batches
relevantapimethods: CreateBatches
date: 2018-04-09 17:00
author: John Horsley
comments: true
categories: [avatax, batches]
product: blog
doctype: blog
disqus: 1
---
<h2>Process transactions in bulk - is it right for your organization?</h2>

Avalara offers two options for processing your organization’s transactions – one at a time through the <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Transactions/CreateTransaction/" target="_blank">CreateTransaction</a> end point or in bulk through the <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Batches/CreateBatches/" target="_blank">CreateBatches</a> end point. Which option is the best fit for your organization? Let's talk about what each of them and what they mean! 

<h2>CreateBatches End Point</h2>

When you use the <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Batches/CreateBatches/" target="_blank">CreateBatches</a> end point you can send us one large file of transations for our tax engine to process for you. While this will not give you real time tax information on your invoices, this will allow you to send a large amount of invoices to us and then use <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Batches/DownloadBatch" target="_blank">DownloadBatch</a> to get your results of the batch file you have processed. This options is best if your transaction workflow includes estimates on your invoices and batch invoices for finalization in bulk later on. 

One key thing to remember is that batches are processed as they come in but do not carry the <a href="https://developer.avalara.com/avatax/service-availability-and-performance/" target="_blank">service level guarantee</a> of our <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Transactions/CreateTransaction/" target="_blank">CreateTransaction</a> on demand calculation service. For best results you will want to use the <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Transactions/CreateTransaction/" target="_blank">CreateTransaction</a> end point.

<h2>CreateTransaction End Point</h2>

With the <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Transactions/CreateTransaction/" target="_blank">CreateTransaction</a> end point, you can send transactions to us as you create them within your transaction workflow. This allows you to get real time tax information for your system and posting of your invoices to the admin console. 

Which option best fits in with your workflow? If you need real time calculation then <a href="https://developer.avalara.com/api-reference/avatax/rest/v2/methods/Transactions/CreateTransaction/" target="_blank">CreateTransaction</a> is for you! If you're processing multiple transactions after hours all at once and do not need real-time calculations back into your system, then processing them through batches is the way to go!


<BR>
<BR>