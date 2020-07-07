---
name: Considdr Insights Chrome Extension
tags: ["Javascript (ES6)", "React", "Webpack", "Chrome API", "Ruby on Jets", "AWS Lambda", "AWS API Gateway"]
github_link: "https://github.com/Considdr/considdr-insights-chrome-extension"
date: 2020-06-16
summary: "A Chrome extension that highlights the key insights in documents across the web."
intro: "A Chrome extension that highlights the key insights in documents across the web. The extension interfaces with the Considdr API to retrieve insights for a specific page. The user can either manually highlight insights through a button click, or, if they enable it, automatically highlight insights as they visit new pages."
---

{% include image.html url="/_images/considdr-insights.gif" description="Highlighting insights across documents" %}

# Intention

Due to the COVID-19 quarantine, I had a lot of extra free time on my hands, and I wanted to do something productive with it. A project that we have wanted to do for a long time at [Considdr](https://www.considdr.com), one that received significant enthusiasm from our customers, was to build a Chrome extension that highlights key insights that we have identified from articles as our customers browse the web. This way, when reading new articles or sources that were relevant to them, they could quickly and easily pick out the high quality insights. I took it upon myself to build this extension **1.** to build a product that would significantly improve our customers research experience and **2.** to learn something new now that I had the time.

This project was more of a proof of concept that we wanted to test with our customers to see if it actually worked and helped them in the way that we were intending it to.

# Execution / Design Decisions

This project consisted of two parts, the Chrome extension itself (the frontend) and the insights API (the backend) that it interfaces with to get insights on a particular article.

## Insights API

The Insights API is a web API that was developed to support the Chrome extension. This API returns the key insights that Considdr has identified from a particular article on the web. The API code is proprietary and unfortunately not available on GitHub.

### AWS Lambda and API Gateway

To build this API I chose to use AWS Lambda and AWS API Gateway. Lambda is a serverless computing platform that executes code only when needed (it's event-driven). API Gateway is an API management tool that sits between a client and a backend service, routing incoming requests to the right places. Lambda and API Gateway actually work very well in tandem, making it very easy to build a REST API. API Gateway sets up the HTTP endpoints for the API, and then routes incoming requests to the corresponding Lambda function which then services the request. I chose to use Lambda as its quick and easy to set up (no need to provision any servers), and you only pay for the compute time that you use.

### Ruby on Jets

I decided to use the [Ruby on Jets](https://rubyonjets.com/) framework to develop this API. Jets is a Ruby serverless framework that makes deploying serverless services to AWS extremely easy, turning your code into AWS resources. AWS can get overwhelming at times when creating and dealing with a lot of resources, and does require a lot of configuration, often in the format of YAML files. I decided to let Jets handle a lot of the AWS configuration and to focus on building the API itself.

Jets works a lot like Ruby on Rails. However, as Jets creates serverless resources, routes get turned into API Gateway endpoints, and controller functions get turned in Lambda functions. Thus, whenever one of the API Gateway endpoints is hit, it executes the correponding Lambda function to service the request. I have a lot of experience working with Rails at Considdr, and so working with Jets was seamless and very enjoyable.

### Lambda Authorizers

Since we wanted to restrict access to this API to our customers, we needed user authentication. API Gateway offers a really neat feature to control access to an API, in the form of [Lambda Authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html). A Lambda authorizer is a Lambda function that, using a custom authorization scheme, is run before executing an API method to authenticate the incoming request. If the request is authorized, the endpoint function is executed, if not, the request is rejected. 

The authorizer returns an IAM policy as output to indicte if the request was authorized or not. The policy will look something like the following:

{% highlight ruby %}

{
  "principalId": user_id, # The id of the current user
  "policyDocument": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Action": "execute-api:Invoke",
        "Effect": "Allow", # or Deny if the request was not authorized
        "Resource": resource # i.e. the correponding Lambda function
      }
    ]
  }
}

{% endhighlight %}

Lambda authorizers help to separate and centralize authentication logic into a single function so that you don't need to package it with every function. Furthermore, you can cache the authorizer response for a specific user to speed up subsequent requests.

There are two types of Lambda authorizers:
* Token-based authorizers: Authorizes the request using a provided bearer token.
* Request parameter-based authorizers: Authorizes the request using a combination of headers, query string parameters, stageVariables, and $context variables.

I chose to build a Request parameter-based Lambda Authorizer as I chose to authenticate users using a token stored in a cookie on the user's browser. When a user requests insights from the API, the Lambda Authorizer will first check the validity of the request's cookies before authorizing the request.

### Authentication

I went back and forth a lot of how I was going to design the authentication for this Chrome Extension and API. Ultimately I settled on using Token-Based Authentication and storing the token in a cookie on the user's browser.

<!-- WHY TOKEN -->

<!-- ACCESS TOKEN -->

#### JWT

The token that is stored in the cookie set by the server is a [JSON Web Token](https://jwt.io/introduction/) (JWT). A JWT is a standardized, compact and digitally signed container format used to securely transfer information between parties. A JWT is cryptographically signed, meaning that it cannot be modified without invalidating it.

A JWT consists of three parts:
* Header: A JSON Object containing the JWT metadata, including the cryptographic algorithms to generate the signature. This is Base64Url encoded to form the first part of the JWT.
* Payload: A JSON Object containing a set of claims (i.e. data), e.g. user information. This is Base64Url encoded to form the second part of the JWT.
* Signature: A string that securely validates the token, ensuring that the JWT has not been tampered with. This signature is generated using the chosen cryptographic algorithm and a secret key. This forms the third part of the JWT.

This is encoded and compacted into three Base64-URL strings separated by dots, generating the JWT itself, for example:

{% highlight ruby %}

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

{% endhighlight %}

This token can then be securely transfered and decoded (you will need the correct secret key to do so) to extract the payload.

I chose to use JWTs as they are a stateless (no need to store a session in the database as the user information is store in the token itself) and secure way to automatically authenticate the user.

Below is an example of the JWT structure that would be generated and set in the cookie in response to a sign in request to the API:

{% highlight ruby %}

JWT.encode(
    {
        user_id: user_id, # The current user's id
        exp: expiration, # This JWT's expiration UNIX timestamp
        jwt_version: jwt_version # This JWT's version number
    },
    ENV['SECRET_KEY_BASE'], # The secret key
    'HS256' # My chosen cryptographic algorithm
)

{% endhighlight %}

When a request is made to the API, the Lambda Authorizer will check the validity of the provided JWT before authorizing the request.

As a JWT is theoretically infinitely valid if not tampered with, there needs to be some mechanism to invalidate it. The mechanism I chose to implement was to set an expiration time in the JWT payload, after which, the server would reject it. I also chose to include a JWT version number so that once a user signs out from the Chrome extension, the version number is incremented, and so that JWT will no longer be valid.

#### Cookies

A [cookie](https://en.wikipedia.org/wiki/HTTP_cookie) is a small piece of data that is sent from a remote server and stored in the user's browser. This data can be used to aid in authentication, personalization, etc. If using cookies to aid in authentication, when a user hits the sign-in endpoint and provides their log in credentials, the server will set a cookie in the user's browser, containing some authentication key, in its response. Cookies get set if the server HTTP response contains the `Set-Cookie` header. Then, on every subsequent request, the browser will include the cookie in its request to the server, and the server checks the validity of the cookie value, automating the authentication of every request.

I chose to store the token in a cookie as I wanted to protect against a common security risk, Cross-site Scripting (XSS) attacks. When implementing token-based authentication, the access token needs to be stored somewhere on the client. 

An XSS attack is a malicious code injection

CSFR

After signing in on the Chrome extension (i.e. hitting the log in endpoint of the API), the API response will set a cookie in the user's browser. This cookie will be sent along with every subsequent request to the Insights API, allowing them to be authorized by the Lambda Authorizer, gaining them access to the Considdr insights.

Using cookies works best for when the client is a browser and in the case of the Chrome extension, the client is a browser. However, for clients besides browsers, managing cookies is very inconvenient. Thus, if we choose to expand the functionality of our API, we may need to expand its authentication capabilities to make it convenient to interface with it using a variety of clients.

## Chrome Extension

The other half of this project was building the Chrome extension itself.

The extension allows our customers to sign in, and then it will highlight the key insights that Considdr has identified from articles as they browse the web. A user can either trigger the highlighting manually by cliking a button in the extension which will highlight the insights on the current article, or, if they enable automatic highlighting, the extension will automatically highlight key insights as they browse articles. The number of insights that were found in a specific article is reflected in the extension badge.

I used the following boilerplate to build this extension: https://github.com/samuelsimoes/chrome-extension-webpack-boilerplate

### Chrome API

The way that a Chrome Extension interfaces with Chrome itself is through the [Chrome API](https://developer.chrome.com/extensions/api_index).

### Background Scripts / Popup

<!-- Background.js -->

<!-- Storage -->

<!-- Message Passing -->

### Content Script

<!-- Content Script -->

### React / Webpack

<!-- Simple summary of using react & webpack -->

# Future Work

This project has an abundance of possible extensions that would make it much better. These include but are not limited to:
* Implementing a send feature where the user could send the current page that they are on to be processed by Considdr so that key insights could be identified. Considdr does not currently scrape the entire web, and we do not have every article parsed. This way, a user could send an article to Considdr that has not already been scraped, so that insights could be identified for them.
* Allowing users to manually highlight insights on a page and save to them to their Considdr collections for future reference and use
* Leveraging Considdr's Similar Insights feature to show users similar insights from articles across the web if they hover over an identified insight. Considdr groups insights it has identified based on similarity using NLP. So if a group of insights are making the same or a very similar argument, they are clustered together. This way, users of Considdr can easily identify trends in a space of interest, it reduces the need to read the same thing written differently again and again, and can also increase one's confidence that the point being made is valid. This extension can utilize this feature by showing users the similar insights of the insights identified of a page, and direct them to the sources of those other insights.
* Expand the authentication capabilities of the Insights API to account for a variety of clients
