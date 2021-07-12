---
title: "Complete Guide to CSRF"
categories: [craft]
date: 2021-06-14 06:00:00 +1000
modified: 2021-06-13 06:00:00 +1000
author: pratikdas
excerpt: "In this article, we will understand cross-origin resource sharing (CORS), different types of CORS requests, CORS headers, and describe some common examples of security vulnerabilities caused by CORS misconfigurations along with best practices for secure CORS implementations."
image:
  auto: 0074-stack
---

Cross-site request forgery, also called CSRF or XSRF, is a type of web security vulnerability. With a successful CSRF attack, an attacker can impersonate an authenticated user in an website and perform malicious actions without their knowledge. 

CSRF vulnerablities can be easily exploited causing loss of customer confidence on the targetted website, and even result in fraud or theft of financial resources.

In this article, we will understand :
- What constitutes a Cross-site Request Forgery (CSRF) attack
- What makes Websites vulnerable to a CSRF attack
- What are some methods to secure Websites from CSRF attack

{% include github-project.html url="https://github.com/thombergs/code-examples/tree/master/cors" %}

## What is CSRF?

CSRF attacks websites which trust some form of authentication by users before they perform any actions. Attackers exploit this trust of the website on an authenticated user and send forged requests on behalf of the user. An example will make this more clear:

Let us suppose an user Alice logs in to a website xyz.com which is vulnerable to a CSRF attack. After authentication with her credentials the user can make payments to different parties added by the her. She does this by filling up some fields and clicking a URL: xyz.com?toAccount=yyyyy.

The attacker can create a forged request of the form xyz.com?toAccount=attacker's-AccountNumber and send this URL in an email to Alice with an inducement like "click here to win 1000 USD". If Alice clicks this link in the same browser, the request will go through and the amount will get transferred to the attacker's account.

This example although very trivial can be extended to scenarios where an attacker can performing other actions with more damaging potential like changing Alice's password and registered email which will block her access completely.

Attackers often use social engineering websites to launch a CSRF attack by sending maliciously crafted URLs with attractive inducements. When the victim user clicks this URL, the user’s browser then sends the maliciously crafted request to the targeted Web application. The request will be processed, if the Web application is vulnerable to CSRF attack.

CSRF is considered a flaw under the [A5 category]((https://owasp.org/www-community/attacks/csrf)) in the OWASP Top ten Web Application Security Risks.

The OWASP website defines CSRF as:
> Cross-Site Request Forgery (CSRF) is an attack that forces an end user to execute unwanted actions on a web application in which they’re currently authenticated. With a little help of social engineering (such as sending a link via email or chat), an attacker may trick the users of a web application into executing actions of the attacker’s choosing.

## How does CSRF work?
After understanding the extent of damage a CSRF attack can cause, let us understand how CSRF works in greater detail.

There are many ways for an attacker to try and exploit the CSRF vulnerability. An attacker constructs CSRF attack through the follwing steps:

### Identify Vulnerable site
A CSRF attack targets Web applications which do not have mechanisms to differentiate between valid requests and forged requests controlled by attacker. 

### Build an exploit URL
The attacker must build an exploit URL.

### Create an inducement
The attacker must also trick Bob into clicking the exploit URL.
Send the inducement through an email

### Active Session


To give an example, let’s say that Bob has an online banking account on samplebank.com. He regularly visits this site to conduct transactions with his friend Alice. Bob is unaware that samplebank.com is vulnerable to CSRF attacks. Meanwhile, an attacker aims to transfer $5,000 from Bob’s account by exploiting this vulnerability. To successfully launch this attack:


The attacker must also trick Bob into clicking the exploit URL.
Bob needs to have an active session with samplebank.com.
Let’s say that the online banking application is built using the GET method to submit a transfer request. As such, Bob’s request to transfer $500 to Alice (with account number 213367) might look like this:

GET https://samplebank.com/onlinebanking/transfer?amount=500&accountNumber=213367 HTTP/1.1

Aligning with the first requirement to successfully launch a CSRF attack, an attacker must craft a malicious URL to transfer $5,000 to the account 425654:

https://samplebank.com/onlinebanking/transfer?amount=5000&accountNumber=425654

Using various social engineering attack methods, an attacker can trick Bob into loading the malicious URL. This can be achieved in various ways. For instance, including malicious HTML image elements onto forms, placing a malicious URL on pages that are often accessed by users while logged into the application, or by sending a malicious URL through email.

The following is an example of a disguised URL:

<img src=“https://samplebank.com/onlinebanking/transfer?amount=5000&accountNumber=425654” width=“0” height=“0”>

Consider the scenario that includes an image tag in an attacker-crafted email to Bob. Upon receiving it, Bob’s browser application opens this URL automatically—without human intervention. As a result, without Bob’s permission, a malicious request is sent to the online banking application. If Bob has an active session with samplebank.com, the application would treat this as an authorized amount transfer request coming from Bob. It would then transfer the amount to the account specified by an attacker.

There are some limitations. To carry out a successful CSRF attack, consider the following:

The success of a CSRF attack depends on a user’s session with a vulnerable application. The attack will only be successful if the user is in an active session with the vulnerable application.
An attacker must find a valid URL to maliciously craft. The URL needs to have a state-changing effect on the target application.
An attacker also needs to find the right values for the URL parameters. Otherwise, the target application might reject the malicious request.

## Preventing CSRF attacks
To defeat a CSRF attack, applications need a way to determine if the HTTP request is legitimately generated via the application’s user interface. The best way to achieve this is through a CSRF token. A CSRF token is a secure random token (e.g., synchronizer token or challenge token) that is used to prevent CSRF attacks. The token needs to be unique per user session and should be of large random value to make it difficult to guess.

A CSRF secure application assigns a unique CSRF token for every user session. These tokens are inserted within hidden parameters of HTML forms related to critical server-side operations. They are then sent to client browsers.

It is the application team’s responsibility to identify which server-side operations are sensitive in nature. The CSRF tokens must be a part of the HTML form—not stored in session cookies. The easiest way to add a non-predictable parameter is to use a secure hash function (e.g., SHA-2) to hash the user’s session ID. To ensure randomness, the tokens must be generated by a cryptographically secure random number generator.

Whenever a user invokes these critical operations, a request generated by the browser must include the associated CSRF token. This will be used by the application server to verify the legitimacy of the end-user request. The application server rejects the request if the CSRF token fails to match the test.
## Common CSRF vulnerabilities
## Defenses against CSRF
Logoff when you are done using a site!
Use multiple browsers, E.g.
One for accessing sensitive sites/applications One for surfing freely
Session time outs
After some period of inactivity, logoff the user
Confirmation pages
Are you sure you want to transfer $1000?
CAPTCHA
Add Session-related information to URLs
Makes it extremely difficult for an attacker to know/predict the structure of the URLs to attack
Random, One-time tokens in forms
## Conclusion

CSRF attacks amount to a large percentage of web-based attacks. Fortunately, it is easy to prevent and thwart attacks before they even happen.

I hope this guide will help you to get started with implementing CORS securely and fixing CORS errors.

You can refer to all the source code used in the article on [Github](https://github.com/thombergs/code-examples/tree/master/cors).
