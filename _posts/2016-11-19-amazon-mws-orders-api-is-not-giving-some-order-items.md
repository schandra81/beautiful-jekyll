---
layout: post
published: true
title: Amazon MWS Orders API is not giving some Order Items
subtitle: >-
  Unable to construct MarketplaceWebServiceOrders_Model_ListOrderItemsResponse
  from provided XML. Make sure that ListOrderItemsResponse is a root element
date: 2016-3-3
---
There is a bug in the Amazon's MWS PHP Client libraries.  Their client library attempted to divide the response from their server by looking for two blank lines. The HTTP Header is separated from the XML body by two newlines, so this generally works. But the PHP function that their client library used was also dividing up the XML response wherever there were two blank lines in the body of the XML as well.

An example helps to explain. A typical response to the ListOrderItems API call would look something like this:

```
HTTP/1.1 200 OK
Date: Wed, 18 Feb 2015 23:06:30 GMT
Server: AmazonMWS
Content-Type: text/xml
Content-Length: 1855
Vary: User-Agent

<?xml version="1.0"?>
<ListOrderItemsResponse xmlns="https://mws.amazonservices.com/Orders/2013-09-01">
  <ListOrderItemsResult>
    <OrderItems>
      <OrderItem>
        <OrderItemId>010203012030</OrderItemId>
        <GiftWrapPrice>
          <Amount>3.49</Amount>
          <CurrencyCode>USD</CurrencyCode>
        </GiftWrapPrice>
         ... other Order Item stuff like price, quantity, etc goes here ...
        <QuantityOrdered>1</QuantityOrdered>
        <SellerSKU>EV-abcd-1234</SellerSKU>
        <Title>My Product Name</Title>
        <GiftMessageText>Happy Birthday,

From: Dad</GiftMessageText>
      </OrderItem>
    </OrderItems>
    <AmazonOrderId>112-0987483-4938494</AmazonOrderId>
  </ListOrderItemsResult>
```
Note that there is a whole blank line between the HTTP headers and the start of the XML response. This is normal and how a client is supposed to separate the header from the body of the response. Note also that there is a whole blank line in the middle of the GiftMessageText. The PHP function that the client library was using was also splitting up the response body there, so that the body only contained through the text Happy Birthday, and didn’t include the rest of the response. Without the other half of the response, the whole XML response was treated as invalid, and nothing would continue.

You have to fix the MWS PHP Libraries by simply adding two characters, which tells the code to just split on the first occurrence of the two newlines. This correctly parses the response and makes everything work normal.

simply modify line 687 in Client.php. Change

```
$responseComponents = preg_split("/(?:\r?\n){2}/", $response);
```

to

```
$responseComponents = preg_split("/(?:\r?\n){2}/", $response, 2);
```
