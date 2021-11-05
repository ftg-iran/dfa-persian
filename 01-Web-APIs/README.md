<div dir="rtl">

# Web APIs

Before we start building our own web APIs it’s important to review how the web really works.
After all, a “web API” literally sits on top of the existing architecture of the world wide web and
relies on a host of technologies including HTTP, TCP/IP, and more.

In this chapter we will review the basic terminology of web APIs: endpoints, resources, HTTP
verbs, HTTP status codes, and REST. Even if you already feel comfortable with these terms, I
encourage you to read the chapter in full.

### World Wide Web

The Internet is a system of interconnected computer networks that has existed since at least the
1960s11 . However, the internet’s early usage was restricted to a small number of isolated networks,
largely government, military, or scientific in nature, that exchanged information electronically.
By the 1980s, many research institutes and universities were using the internet to share data.
In Europe, the biggest internet node was located at CERN (European Organization for Nuclear
Research) in Geneva, Switzerland, which operates the largest particle physics laboratory in the
world. These experiments generate enormous quantities of data that need to be shared remotely
with scientists all around the world.

Compared with today, though, overall internet usage in the 1980s was miniscule. Most people
did not have access to it or even understood why it mattered. A small number of internet
nodes powered all the traffic and the computers using it were primarily within the same, small
networks.

This all changed in 1989 when a research scientist at CERN, Tim Berners-Lee, invented HTTP
and ushered in the modern World Wide Web. His great insight was that the existing hypertext12 system, where text displayed on a computer screen contained links (hyperlinks) to other
documents, could be moved onto the internet.

His invention, Hypertext Transfer Protocol (HTTP)13 , was the first standard, universal way to
share documents over the internet. It ushered in the concept of web pages: discrete documents
with a URL, links, and resources such as images, audio, or video.

Today, when most people think of “the internet,” they think of the World Wide Web, which is now
the primary way that billions of people and computers communicate online.

### URLs

A URL (Uniform Resource Locator) is the address of a resource on the internet. For example, the
Google homepage lives at https://www.google.com.

When you want to go to the Google homepage, you type the full URL address into a web browser.
Your browser then sends a request out over the internet and is magically connected (we’ll cover
what actually happens shortly) to a server that responds with the data needed to render the
Google homepage in your browser.

This request and response pattern is the basis of all web communication. A client (typically a
web browser but also a native app or really any internet-connected device) requests information
and a server responds with a response.

Since web communication occurs via HTTP these are known more formally as HTTP requests
and HTTP responses.

Within a given URL are also several discrete components. For example, consider the Google
homepage located at https://www.google.com. The first part, https, refers to the scheme used.
It tells the web browser how to access resources at the location. For a website this is typically
http or https, but it could also be ftp for files, smtp for email, and so on. The next section,
www.google.com, is the hostname or the actual name of the site. Every URL contains a scheme
and a host.

Many webpages also contain an optional path, too. If you go to the homepage for Python at https://www.python.org and click on the link for the “About” page you’ll be redirected to
https://www.python.org/about/. The /about/ piece is the path.

In summary, every URL like https://python.org/about/ has three potential parts:

- a scheme - https
- a hostname - www.python.org
- and an (optional) path - /about/

### Internet Protocol Suite

Once we know the actual URL of a resource, a whole collection of other technologies must work
properly (together) to connect the client with the server and load an actual webpage. This is
broadly referred to as the internet protocol suite14 and there are entire books written on just
this topic. For our purposes, however, we can stick to the broad basics.

Several things happen when a user types https://www.google.com into their web browser and
hits Enter. First the browser needs to find the desired server, somewhere, on the vast internet.
It uses a domain name service (DNS) to translate the domain name “google.com” into an IP
address15 , which is a unique sequence of numbers representing every connected device on the
internet. Domain names are used because it is easier for humans to remember a domain name
like “google.com” than an IP address like “172.217.164.68”.

After the browser has the IP address for a given domain, it needs a way to set up a consistent
connection with the desired server. This happens via the Transmission Control Protocol (TCP)
which provides reliable, ordered, and error-checked delivery of bytes between two application.

To establish a TCP connection between two computers, a three-way “handshake” occurs between the client and server:

- The client sends a SYN asking to establish a connection
- The server responds with a SYN-ACK acknowledging the request and passing a connection
parameter
- The client sends an ACK back to the server confirming the connection

Once the TCP connection is established, the two computers can start communicating via HTTP.

### HTTP Verbs

Every webpage contains both an address (the URL) as well as a list of approved actions known as
HTTP verbs. So far we’ve mainly talked about getting a web page, but it’s also possible to create,
edit, and delete content.

Consider the Facebook website. After logging in, you can read your timeline, create a new
post, or edit/delete an existing one. These four actions Create-Read-Update-Delete are known
colloquially as CRUD functionality and represent the overwhelming majority of actions taken
online.

The HTTP protocol contains a number of request methods16 that can be used while requesting
information from a server. The four most common map to CRUD functionality. They are POST,
GET, PUT, and DELETE.

<div dir="ltr">

```
CRUD                            HTTP Verbs
----
Create  <-------------------->  POST
Read    <-------------------->  GET
Update  <-------------------->  PUT
Delete  <-------------------->  DELETE
```

</div>

To create content you use POST, to read content GET, to update it PUT, and to delete it you use
DELETE.

### Endpoints

A website consists of web pages with HTML, CSS, images, JavaScript, and more. But a web API
has endpoints instead which are URLs with a list of available actions (HTTP verbs) that expose data (typically in JSON17 , which is the most common data format these days and the default for
Django REST Framework).

For example, we could create the following API endpoints for a new website called mysite.

<div dir="ltr">

```
https://www.mysite.com/api/users      # GET returns all users
https://www.mysite.com/api/users/<id> # GET returns a single user
```

</div>

In the first endpoint, /api/users, an available GET request returns a list of all available users. This type of endpoint which returns multiple data resources is known as a collection.

The second endpoint /api/users/<id> represents a single user. A GET request returns informa-
tion about just that one user.

If we added POST to the first endpoint we could create a new user, while adding DELETE to the
second endpoint would allow us to delete a single user.

We will become much more familiar with API endpoints over the course of this book but
ultimately creating an API involves making a series of endpoints: URLs with associated HTTP
verbs.

A webpage consists of HTML, CSS, images, and more. But an endpoint is just a way to access data
via the available HTTP verbs.

### HTTP

We’ve already talked a lot about HTTP in this chapter, but here we will describe what it actually
is and how it works.
HTTP is a request-response protocol between two computers that have an existing TCP connec-
tion. The computer making the requests is known as the client while the computer responding
is known as the server. Typically a client is a web browser but it could also be an iOS app or really
any internet-connected device. A server is a fancy name for any computer optimized to work over the internet. All we really need to transform a basic laptop into a server is some special
software and a persistent internet connection.

Every HTTP message consists of a status line, headers, and optional body data. For example, here
is a sample HTTP message that a browser might send to request the Google homepage located
at https://www.google.com.

<div dir="ltr">

```
GET / HTTP/1.1
Host: google.com
Accept_Language: en-US
```

</div>

The top line is known as the request line and it specifies the HTTP method to use (GET), the path
(/), and the specific version of HTTP to use (HTTP/1.1).

The two subsequent lines are HTTP headers: Host is the domain name and Accept_Language is
the language to use, in this case American English. There are many HTTP headers18 available.

HTTP messages also have an optional third section, known as the body. However we only see a
body message with HTTP responses containing data.

For simplicity, let’s assume that the Google homepage only contained the HTML “Hello, World!”
This is what the HTTP response message from a Google server might look like.

<div dir="ltr">

```
HTTP/1.1 200 OK
Date: Mon, 03 Aug 2020 23:26:07 GMT
Server: gws
Accept-Ranges: bytes
Content-Length: 13
Content-Type: text/html; charset=UTF-8

Hello, world!
```

</div>

The top line is the response line and it specifies that we are using HTTP/1.1. The status code 200
OK indicates the request by the client was successful (more on status codes shortly).

The next eight lines are HTTP headers. And finally after a line break there is our actual body
content of “Hello, world!”.

Every HTTP message, whether a request or response, therefore has the following format:

<div dir="ltr">

```
Response/request line
Headers...

(optional) Body
```

</div>

Most web pages contain multiple resources that require multiple HTTP request/response cycles.
If a webpage had HTML, one CSS file, and an image, three separate trips back-and-forth between
the client and server would be required before the complete web page could be rendered in the
browser.

### Status Codes

Once your web browser has executed an HTTP Request on a URL there is no guarantee things will
actually work! Thus there is a quite lengthy list of HTTP Status Codes19 available to accompany
each HTTP response.

You can tell the general type of status code based on the following system:

- 2xx Success - the action requested by the client was received, understood, and accepted
- 3xx Redirection - the requested URL has moved
- 4xx Client Error - there was an error, typically a bad URL request by the client
- 5xx Server Error - the server failed to resolve a request

There is no need to memorize all the available status codes. With practice you will become
familiar with the most common ones such as 200 (OK), 201 (Created), 301 (Moved Permanently),
404 (Not Found), and 500 (Server Error).

The important thing to remember is that, generally speaking, there are only four potential
outcomes to any given HTTP request: it worked (2xx), it was redirected somehow (3xx), the client
made an error (4xx), or the server made an error (5xx).

These status codes are automatically placed in the request/response line at the top of every
HTTP message.

### Statelessness

A final important point to make about HTTP is that it is a stateless protocol. This means
each request/response pair is completely independent of the previous one. There is no stored
memory of past interactions, which is known as state20 in computer science.

Statelessness brings a lot of benefits to HTTP. Since all electronic communication systems have
signal loss over time, if we did not have a stateless protocol, things would constantly break if
one request/response cycle didn’t go through. As a result HTTP is known as a very resilient
distributed protocol.

The downside however is that managing state is really, really important in web applications. State
is how a website remembers that you’ve logged in and how an e-commerce site manages your
shopping cart. It’s fundamental to how we use modern websites, yet it’s not supported on HTTP
itself.

Historically state was maintained on the server but it has moved more and more to the client,
the web browser, in modern front-end frameworks like React, Angular, and Vue. We’ll learn more
about state when we cover user authentication but remember that HTTP is stateless. This makes
it very good for reliably sending information between two computers, but bad at remembering
anything outside of each individual request/response pair.

### REST

REpresentational State Transfer (REST)21 is an architecture first proposed in 2000 by Roy Fielding

in his dissertation thesis. It is an approach to building APIs on top of the web, which means on
top of the HTTP protocol.

Entire books have been written on what makes an API actually RESTful or not. But there are three
main traits that we will focus on here for our purposes. Every RESTful API:

- is stateless, like HTTP
- supports common HTTP verbs (GET, POST, PUT, DELETE, etc.)
- returns data in either the JSON or XML format

Any RESTful API must, at a minimum, have these three principles. The standard is important
because it provides a consistent way to both design and consume web APIs.

### Conclusion

While there is a lot of technology underlying the modern world wide web, we as developers
don’t have to implement it all from scratch. The beautiful combination of Django and Django
REST Framework handles, properly, most of the complexity involved with web APIs. However it
is important to have at least a broad understanding of how all the pieces fit together.

Ultimately a web API is a collection of endpoints that expose certain parts of an underlying
database. As developers we control the URLs for each endpoint, what underlying data is available,
and what actions are possible via HTTP verbs. By using HTTP headers we can set various levels
of authentication and permission too as we will see later in the book.

</div>