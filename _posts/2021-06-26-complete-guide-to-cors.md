---
title: "Complete Guide to CORS"
categories: [craft]
date: 2021-06-14 06:00:00 +1000
modified: 2021-06-13 06:00:00 +1000
author: pratikdas
excerpt: "CORS is a protocol and security standard for browsers that helps to maintain the integrity of a website and secure it from unauthorized access. In this article, we will understand cross-origin resource sharing (CORS) and describe some common examples of security vulnerabilities caused by CORS misconfigurations along with best practices for secure CORS implementations."
image:
  auto: 0074-stack
---

“CORS” stands for **C**ross-**O**rigin **R**esource **S**haring. CORS is a protocol and security standard for browsers that helps to maintain the integrity of a website and secure it from unauthorized access. 

It enables JavaScripts running in browsers to connect to APIs and other web resources like fonts, and stylesheets from multiple different providers.

In this article, we will understand the following aspects of CORS:
- What's the CORS standard?
- What are the different types of CORS requests?
- What are different CORS headers and what do we need them for?
- What security vulnerabilities exist around cross-origin requests?
- What are the best practices for secure CORS implementations?

{% include github-project.html url="https://github.com/thombergs/code-examples/tree/master/cors" %}


## What is CORS?
**CORS is a security standard implemented by browsers that enable scripts running in browsers to access resources located outside of the browser's domain.** 

The CORS policy is published under the [Fetch standard](https://fetch.spec.whatwg.org/#http-cors-protocol) defined by the [WHATWG](https://whatwg.org) community which also publishes many web standards like [HTML5](https://html.spec.whatwg.org/multipage/),[DOM](https://dom.spec.whatwg.org), and [URL](https://url.spec.whatwg.org).

According to the Fetch standard spec:

> The CORS protocol consists of a set of headers that indicates whether a response can be shared cross-origin. For requests that are more involved than what is possible with HTML’s form element, a CORS-preflight request is performed, to ensure the request’s current URL supports the CORS protocol.

Some scenarios of browsers fetching resources where CORS comes into play are:

- Display a map of a user's location in an HTML or single page application hosted in a domain xyz.com by calling google's Map API `https://maps.googleapis.com/maps/api/js`.
- Show tweets from a public Twitter handle in an HTML hosted in a domain xyz.com by calling a Twitter API `https://api.twitter.com/xxx/tweets/xxxxx`
- Using web fonts like [Typekit](https://fonts.adobe.com/typekit) and [Google Fonts](https://fonts.googleapis.com) in an HTML hosted in a domain xyz.com from their remote domains.

Let us understand in greater detail the role of a CORS policy for fetching resources from remote origins, followed by how CORS policy is enforced by browsers, and how we implement CORS in our applications in the subsequent sections.

## Relaxation of the Same-Origin Policy

The role of a CORS policy is to maintain the integrity of a website and secure it from unauthorized access. 

The CORS protocol was defined to relax the default security policy called the Same-Origin Policy (SOP) used by the browsers to protect their resources. 

**The Same-Origin Policy permits the browser to load resources only from a server hosted in the same origin as the browser.** 

The SOP was defined in the early years of the web and turned out to be too restrictive for the new age applications where we often need to fetch different kinds of resources from multiple origins.

The CORS protocol is implemented by all modern browsers to allow controlled access to resources located outside of the browser's origin. 

## CORS Terminology
Before going further, let us define some frequently used terms like browsers, servers, origins, cross-origins. We will then use these terms consistently throughout this article.

### What is an Origin?

An Origin in the context of CORS consists of three elements:

- URI scheme, for example `http://` or `https://`
- Hostname like `www.xyz.com`
- Port number like `8000` or `80` (default HTTP port)

**We consider two URLs to be of the same Origin only if all three elements match**.

A more elaborate explanation of the Web Origin Concept is available in [RFC 6454](https://tools.ietf.org/html/rfc6454).

### Origin Server and Cross-Origin Server
"Origin server" and "Cross-Origin server" are not CORS terms. But we will be using these terms for referring to the server that is hosting the source application and the server to which the browser will send the CORS request. This diagram shows the main participants of a CORS flow:

![cors terms](/assets/img/posts/cors/CORS-terms.png)

The following steps happen, when a user types in a URL: http://www.example.com/index.html in the browser:
 1. The browser sends the request to a server in a domain named `www.example.com`. We will call this server "**Origin server**" which hosts the page named `index.html`.
 2. The "Origin server" returns the page named `index.html` as a response to the browser.
 3. The "Origin server" also hosts other resources like the `movies.json` API in this example.
 4. The browser can also fetch resources from a server in a different domain like `www.xyz.com`. We will call this server "**Cross-Origin server**".
 5. The browser uses Ajax technology with the built-in `XMLHttpRequest` object, or since 2017 the new `fetch` function within JavaScript to load content on the screen without refreshing the page.

These sequence of steps are represented in this sequence diagram:

![cors seq](/assets/img/posts/cors/seq.png)


We will use the terms "Origin server" and "Cross-Origin server" throughout this article. 

The **"Origin server" will refer to the server from which the web page is fetched and "Cross-Origin server" will represent any server that is different from the "Origin server".**

### Same Origin vs Cross Origin
As stated earlier, the Same-Origin Policy (SOP) is a default security policy implemented by browsers. The SOP permits the browser to load resources only from the "Origin server".

In the absence of the Same-Origin Policy, any website will be able to access the document object model (DOM) of other websites and allow it to access potentially sensitive data as well as perform malicious actions on other websites without requiring user consent.

The following figure shows an HTML page `currentPage.html` making same or cross-origin requests to `targetPage.html`: 

![same vs cross origin urls](/assets/img/posts/cors/samevscross.png)

As we can see in this diagram, same-origin requests are allowed and cross-origin requests are blocked by default by the browser. 

The URLs of `targetPage.html` that the browser rendering `currentPage.html` considers to be of the same or cross-origin are listed in this table. The default port is `80` for HTTP and `443` for HTTPS for the URLs in which we have not specified any port:

<style>
.table td {
  padding: 5px
}
</style>

|URLs being Matched| Same Origin or Cross Origin| Reason |
|-|-|-|
|http://www.mydomain.com/targetPage.html|Same Origin|same scheme, host, and port|
|http://www.mydomain.com/subpage/targetPage.html|Same Origin|same scheme, host, and port|
|https://www.mydomain.com/targetPage.html|Cross Origin|same host but different scheme and port|
|http://pg.mydomain.com/targetPage.html|Cross Origin|different host|
|http://www.mydomain.com:8080/targetPage.html|Cross Origin|different port|
|http://pg.mydomain.com/mypage1.html|Cross Origin|different host|
{: .table}


If the origins corresponding to the URLs are same, we can run JavaScripts in `currentPage.html` which can fetch contents from `targetPage.html`.

In contrast, for cross-origin URLs, JavaScripts running in `currentPage.html` will be prevented from fetching contents from `targetPage.html` without a CORS policy configured correctly.

## How Browsers Implement CORS Policy

The CORS protocol is enforced only by the browsers. The browser does this by sending a set of CORS headers to the "Cross-Origin server" which returns specific header values in the response. Based on the header values returned in the response from the "Cross-Origin server", the browser provides access to the response or blocks the access by showing a CORS error in the browser console. 

### Using the Header based Protocol of CORS
When a request for fetching a resource is made from a web page, the browser detects whether the request is to the "Origin Server" or the "Cross-Origin Server" and applies the CORS policy if the request is for the "Cross-Origin Server".

The browser sends a header named `Origin` with the request to the "Cross-Origin server". The "Cross-Origin server" processes this request and sends back a header named `Access-Control-Allow-Origin` in the response. 

The browser checks the value of the `Access-Control-Allow-Origin` header in the response and renders the response only if the value of the `Access-Control-Allow-Origin` header is the same as the `Origin` header sent in the request. 

The "Cross-Origin Server" can also use wild cards like `*` as the value of the `Access-Control-Allow-Origin` header to represent a partial match with the value of the `Origin` header received in the request.

### CORS Failures

CORS failures cause errors but specifics about the error are not available to the browser for security reasons like an attacker can take hints from the error message to tailor subsequent attacks to increase the chances of success. 

The only way to know about the error is by looking at the browser's console for details of the error which is usually in the following form:

`Access to XMLHttpRequest at 'http://localhost:8000/orders' from origin 'http://localhost:9000' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.`

The error displayed in the browser console is accompanied by an error "reason" message. The reason message can differ across browsers depending on the implementation. To get an idea of some reasons behind CORS errors, we can check the error [reason messages for Firefox browser](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS/Errors#cors_error_messages).

### Type of CORS Requests Sent By Browser

The browser determines the type of request to be sent to the "Cross-Origin server" depending on the kind of operations we want to perform with the resource in the "Cross-Origin server". 

The browser can send three types of requests to the "Cross-Origin server": 
- `simple`
- `preflight`
- `requests with credentials`

Let us understand these request types and observe them in the browsers' network log by running an example in the subsequent sections.

#### Simple CORS Requests (GET, POST, and HEAD)
Simple requests are sent by the browser for performing operations it considers safe like a `GET` request for fetching data or a `HEAD` request to check status. The request sent by the browser is simple if one of the below conditions applies: 

- The HTTP request method is `GET`, `POST`, or `HEAD`
- The HTTP request contains a CORS safe-listed header: `Accept`, `Accept-Language`, `Content-Language`, `Content-Type`.
- When the HTTP request contains `Content-Type` header, it contains as it's values: `application/x-www-form-urlencoded`, `multipart/form-data`, or `text/plain`
- No event listeners are registered on any [XMLHttpRequestUpload](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest/upload) object
- No `ReadableStream` object is used in the request

The browser sends the simple request as a normal request similar to the Same Origin request after adding the `Origin` header, and the `Access-Control-Allow-Origin` header is checked by the browser when the response is returned. 

The browser is able to read and render the response only if the value of the `Access-Control-Allow-Origin` header matches the value of the `Origin` header sent in the request. The `Origin` header contains the source origin of the request.


#### Preflight Requests
In contrast to simple requests, the browser sends preflight requests for operations that intend to change anything in the "Cross-Origin server" like an HTTP `PUT` method to update a resource or HTTP `DELETE` for deleting a resource. 

These requests are not considered safe so the web browser first makes sure that cross-origin communication is allowed by first sending a preflight request before sending the actual request to the "Cross-Origin server". Requests which do not satisfy the criteria for simple request also fall under this category.

The preflight request is an HTTP `OPTIONS` method which is sent automatically by the browser to the  "Cross-Origin server", to check that the "Cross-Origin server" will permit the actual request. Along with the preflight request, the browser sends the following headers:

**Access-Control-Request-Method**: This header contains the HTTP method which will be used when the actual request is made.
**Access-Control-Request-Headers**: This is a list of headers that will be sent with the request including any custom headers.
**Origin**: The origin header that contains the source origin of the request similar to the simple request.

The actual request to the "Cross-Origin server" will not be sent if the result of the `OPTIONS` method is that the request cannot be made.

After the preflight request is complete, the actual `PUT` method with CORS headers is sent.

#### CORS Requests with Credentials

In most real-life situations, requests sent to the "Cross-Origin server" need to be loaded with some kind of access credentials which could be an `Authorization` header or cookies. The default behavior of CORS requests is for the requests to be passed without any of these credentials. 

When credentials are passed with the request o the "Cross-Origin server", the browser will not allow access to the response unless the "Cross-origin server" sends a CORS header `Access-Control-Allow-Credentials` with a value of `true`.

## Example of Implementing with CORS in Applications

For observing the CORS requests, let us run two web applications written in Node.Js which will communicate with each other with the CORS protocol:

1. For "Cross-Origin server" we will use a web application named [OrderProcessor](https://github.com/thombergs/code-examples/blob/master/cors/orderprocessor/server.js) that will contain a REST API with `GET` and `PUT` methods.
2. For "Origin server" we will use another web application containing an [HTML page](https://github.com/thombergs/code-examples/blob/master/cors/ecommapp/index.html). We will run JavaScript in this HTML page to communicate with the REST APIs in the `OrderProcessor` application which is our "Cross-Origin server".

We can run these applications in our local machine using `npm` and `node`. The "Origin server" hosting the HTML page is running on `http://localhost:9000`. This makes Ajax calls with the `XMLHttpRequest` object to the `OrderProcessor` application running on the "Cross-Origin server" with URL: `http://localhost:8000` as shown in this figure: 

![cors example](/assets/img/posts/cors/cors-example.png)

These are CORS requests since the HTML in the "Origin server" and `OrderProcessor` application in the  "Cross-Origin server" are running in different Origins (because of different port numbers: 8000 and 9000 although they use the same scheme: HTTP and host: `localhost`).

### Cross-Origin Server Handling CORS Requests in Node.js
Our "Cross-Origin server" is a simple [Node.js](https://nodejs.org/en/) application named `OrderProcessor` built with [Express](https://expressjs.com) framework. We have created two REST APIs in the `OrderProcessor` application with `GET` and `PUT` methods for fetching and updating `orders`. 

This is a snippet of the `GET` method of our `OrderProcessor` application running on "Cross-Origin server" on URL: `localhost:8000`:

```js
app.get('/orders', (req, res) => {
  console.log('Returning orders');
  res.send(orders);
});

```
The `GET` method defined here is used to return a collection of `orders`.

### Client-Side Sending CORS Requests from JavaScript 
For sending requests to the "Cross-Origin server" containing the `OrderProcessor` application, we will use an HTML page and package this inside another Node.js application running on `localhost:9000`. This will be our "Origin Server".

We will call the `GET` and `PUT` methods from this HTML page using the `XMLHttpRequest` JavaScript object:

```html
<html>
<head>
  <script>
       function load(domainURL) {
          var xhttp = new XMLHttpRequest();
          xhttp.onreadystatechange = function() {
              if (this.readyState == 4 && this.status == 200) {
              document.getElementById("demo").innerHTML = this.responseText;
              }
          };
          xhttp.open("GET", domainURL, true);
          xhttp.send();
        }  

        function loadFromCrossOrigin() {
            load("http://localhost:8000/orders")
        } 
  </script>
</head>
<body>            
  <div id="demo">
    <h2>Order Processing</h2>
    <div>
      <button type="button" 
              onclick="loadFromCrossOrigin()">
              ...
      </button>
    </div>
  </div>
</body>
</html>
```
The HTML shown here contains a button which we need to click to trigger the CORS request from the JavaScript method `loadFromCrossOrigin`.

### CORS Error Due to Same Origin Policy
If we run these applications without any additional configurations (setting CORS headers) in the "Cross-Origin server", we will get a CORS error in our browser console as shown below:

![cors failure](/assets/img/posts/cors/cors-fail.png)

This is an error caused by the restriction of accessing cross-origins due to the Same Origin Policy. The error reason is :

```
Access to `XMLHttpRequest` at `http://localhost:8000/orders` from origin `http://localhost:9000` has been blocked by CORS policy: No `Access-Control-Allow-Origin` header is present on the requested resource.`
```

### Fixing the CORS Error For Simple Requests
As suggested in the CORS error description, let us modify the code in the "Cross-Origin server" to return the CORS header `Access-Control-Allow-Origin` in the response:

```js
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "http://localhost:9000");
  next();
});

app.get('/orders', (req, res) => {
  console.log('Returning orders');
  res.send(orders);
});
```
We are returning a CORS header `Access-Control-Allow-Origin` with a value of source origin `http://localhost:9000` to fix the CORS error.

The CORS relevant request headers and response headers from a simple CORS request are shown below:

```
Request URL: http://localhost:8000/orders
Request Method: GET
Status Code: 200 OK

**Request Headers**
Host: localhost:8000
Origin: http://localhost:9000

**Response Headers**
Access-Control-Allow-Origin: http://localhost:9000

```
In this example, the HTML served from `http://localhost:9000` sends a request to the "Cross-Origin server" containing a REST API with the URL `http://localhost:8000/orders`. 

This is a simple CORS request since it is a `GET` request. 

In the browser console log, we can see an `Origin` header sent in the request with a value of `http://localhost:9000` which is the URL of the "Origin server". 

The "Cross-Origin server" responds with a response header `Access-Control-Allow-Origin`.  The browser is able to render the response since the response header `Access-Control-Allow-Origin` has the value `http://localhost:9000` which exactly matches the value of the `Origin` header sent in the request. We can also configure partial matches by using wild cards in the form of `*` or `http://*localhost:9000`.  


### CORS Handling for Preflight Request
Now we will modify our code in the "Cross-Origin server" application to handle preflight request for calls made to the `PUT` method:

```js
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "http://localhost:9000");
  res.header(
    "Access-Control-Allow-Headers",
    "Origin, X-Requested-With, Content-Type, Accept"
  );
  res.header(
    "Access-Control-Allow-Methods",
    "GET, POST, PUT, DELETE"
  );
  next();
});
app.put('/orders', (req, res) => {
  console.log('updating orders');
  res.send(orders);
});

```

For handling the preflight request, we are returning two more headers:
`Access-Control-Allow-Headers` containing the headers `Origin`, `X-Requested-With`, `Content-Type`, `Accept` the server should accept.
`Access-Control-Allow-Methods` containing the HTTP methods `GET`, `POST`, `PUT`, `DELETE` that the browser should send to the server if the preflight request is successful.

When we send the `PUT` request from our HTML page, we can see two requests in the browser network log:

![cors preflight](/assets/img/posts/cors/preflight.png)

The preflight request with the `OPTIONS` method is followed by the actual request with the `PUT` method.

We can observe the following request and response headers of the preflight request in the browser console:

```
Request URL: http://localhost:8000/orders
Request Method: OPTIONS
Status Code: 200 OK
..
..

Request Headers:
Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Access-Control-Request-Headers: content-type
Access-Control-Request-Method: PUT
Connection: keep-alive
Host: localhost:8000
Origin: http://localhost:9000


Response Headers

Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Origin: http://localhost:9000
Allow: GET,HEAD,PUT
```

In this example, the browser served from `http://localhost:9000` sends a `PUT` request to a REST API with URL: `http://localhost:8000/orders`. Since this is a `PUT` request which will change the state of an existing resource in the "Cross-Origin server", the browser sends a preflight request using the HTTP `OPTIONS` method. In response, the "Cross-Origin server" informs the browser that `GET`, `HEAD`, and `PUT` methods are allowed.

### CORS Handling for Request with Credentials
We will now send a credential in the form of a `Authorization` header in our CORS request:

```js
function sendAuthRequestToCrossOrigin() {
    var xhr = new XMLHttpRequest();
    xhr.onreadystatechange = function() {
        if (this.readyState == 4 && this.status == 200) {
          document.getElementById("demo").innerHTML = this.responseText;
        }
    };
    xhr.open('GET', "http://localhost:8000/orders", true);
    xhr.setRequestHeader('Authorization', 'Bearer rtikkjhgffw456tfdd');
    xhr.withCredentials = true;
    xhr.send();
}
```
Here we are sending a bearer token as the value of our `Authorization` header. To allow the browser to read the response, the "Cross-Origin server" needs to send the `Access-Control-Allow-Credentials` header in the response:
```js
app.use(function(req, res, next) {
  res.header("Access-Control-Allow-Origin", "http://localhost:9000");
  res.header(
    "Access-Control-Allow-Headers",
    "Origin, X-Requested-With, Content-Type, Accept, Authorization"
  );
  res.header(
    "Access-Control-Allow-Methods",
    "GET, POST, PUT, DELETE"
  );
  res.header("Access-Control-Allow-Credentials",true);
  next();
});
app.put('/orders', (req, res) => {
  console.log('updating orders');
  res.send(orders);
});

```
We have modified our code in the "Cross-Origin server" to send a value of `true` for the `Access-Control-Allow-Credentials` header so that the browser is able to read the response. We have also added the `Authorization` header in the list of allowed request headers in the header `Access-Control-Allow-Headers`.

We can see the request and response headers in the browser console:
```shell
Request URL: http://localhost:8000/orders
Request Method: GET
Status Code: 200 OK

Response Headers:

Access-Control-Allow-Credentials: true
Access-Control-Allow-Headers: Origin, X-Requested-With, Content-Type, Accept, Authorization
Access-Control-Allow-Methods: GET, POST, PUT, DELETE
Access-Control-Allow-Origin: http://localhost:9000

Request Headers:

Accept: */*
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Authorization: Bearer rtikkjhgffw456tfdd

Origin: http://localhost:9000

```
In this log, we can see the security credential in the form of the `Authorization` header in the request which contains a bearer token. The `Authorization` header is also included in the header named `Access-Control-Allow-Headers` returned from the "Cross-Origin server". The browser can access the response since the value of the `Access-Control-Allow-Credentials` header sent by the server is `true`.

## Vulnerabilities Caused by CORS Misconfiguration
Communications with CORS protocol also have the potential to introduce security vulnerabilities caused by misconfiguration of CORS protocol on the "Cross-Origin server". Some misconfigurations can allow malicious domains to access the API endpoints, while others allow credentials like cookies to be sent from untrusted sources to the "Cross-Origin server" and access sensitive data. 

Let us look at two examples of CORS vulnerabilities caused by any misconfiguration in the code:

### Origin Reflection - Copying the Value of Origin Header in the Response
As we have seen earlier, when the browser sends a request to a "Cross-Origin server", it adds an `Origin` header containing the value of the domain the request originates from. The "Cross-Origin server" needs to return an `Access-Control-Allow-Origin` header with the value of the `Origin` header received in the request. 

There could be a scenario of multiple domains that need access to the resources of the "Cross-Origin server". In that case, the "Cross-Origin server" might set the value of the `Access-Control-Allow-Origin` header dynamically to the value of the domain it receives in the `Origin` header. A Node.js code setting the header dynamically may look like this:

```js
const express = require('express');
const app = express();
...
...
app.get('/orders', (req, res) => {
  
  console.log('Returning orders');

  // set to the value received in Origin header
  res.header("Access-Control-Allow-Origin", req.header('Origin'));
  res.send(orders);
});
```
Here we are reading the value of the `Origin` header received in the request and setting it to the value of the `Access-Control-Allow-Origin` header sent in the response.

Doing this will allow any domain including malicious ones to send requests to the "Cross-Origin server".

### Lenient Regular Expression

Similar to the earlier example, we can check for the value of the `Origin` header in the "Cross-Origin server" code by applying a regular expression. If we want to allow all subdomains to send requests to the "Cross-Origin server", the code will look like this:

```js
const express = require('express');
const app = express();
...
...
app.get('/orders', (req, res) => {
  
  console.log('Returning orders');

  origin = req.getHeader("Origin");
  // allow requests from subdomains of mydomain.com
  let re = new RegExp("https:\/\/[a-z]+.mydomain.com")
  if re.test(origin, regex){
     // set to the value received in Origin header
     res.header("Access-Control-Allow-Origin", origin);
  }
  res.send(orders);
});

```
Since the dot character in the regular expression is not escaped, requests from sites like `https://xyzmydomain.com` will also be served. Any attacker can exploit this vulnerability by buying `xyzmydomain.com` and hosting the malicious code there.

## Avoiding Security Vulnerabilities Caused by CORS Misconfiguration
Here are some of the best practices we can use to implement CORS securely:

1. In the application in the "Cross-Origin server", we can define a whitelist of specific domains that are allowed to access the "Cross-Origin server". When the request arrives, we should validate the `Origin` header against the whitelist to allow or deny access by populating appropriate values in the CORS response headers.
2. Similarly, for the `Access-Control-Allow-Methods` header, we should specify exactly what methods are valid for the whitelisted domains to use. 
3. We should be validating all domains that need to access resources, and the methods other domains are allowed to use if their access request is granted. 
4. We should also use CORS scanners to detect security vulnerabilities caused by CORS misconfigurations.
5. CORS checks should also be part of penetration testing of critical applications. [OWASP guidance on testing CORS](https://owasp.org/www-project-web-security-testing-guide/stable/4-Web_Application_Security_Testing/11-Client-side_Testing/07-Testing_Cross_Origin_Resource_Sharing) provides guidelines for identifying endpoints that implement CORS and ensure the security of the CORS configuration.

## Conclusion
In this article, we learned about CORS and how to use CORS policy to communicate between websites from different origins.

Let us recap the main points that we covered:

1. CORS is a security protocol implemented by browsers that allow us to access resources from a different origin. 
2. CORS requests are of three types: `Simple`, `Preflight`, and `Request with Credentials`.
3. Simple requests are used to perform safe operations like an HTTP `GET` method.
4. Preflight requests are for performing operations with side-affects like `PUT` and `DELETE` methods.
5. Towards the end, we looked at examples of security vulnerabilities caused by CORS misconfigurations and some best practices for secure CORS implementation.

I hope this guide will help you to get started with implementing CORS securely and fixing CORS errors.

You can refer to all the source code used in the article on [Github](https://github.com/thombergs/code-examples/tree/master/cors).
