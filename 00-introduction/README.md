<div dir="rtl">

# Introduction

The internet is powered by RESTful APIs. Behind the scenes even the simplest online task involves
multiple computers interacting with one another.

An API (Application Programming Interface) is a formal way to describe two computers commu-
nicating directly with one another. And while there are multiple ways to build an API, web APIs—
which allow for the transfer of data over the world wide web—are overwhelmingly structured in
a RESTful (REpresentational State Transfer) pattern.

In this book you will learn how to build multiple RESTful web APIs of increasing complexity from
scratch using Django1 and Django REST Framework2 , one of the most popular and customizable
ways to build web APIs, used by many of the largest tech companies in the world including Insta-
gram, Mozilla, Pinterest, and Bitbucket. This approach is also uniquely well-suited to beginners
because Django’s “batteries-included” approach masks much of the underlying complexity and
security risks involved in creating any web API.

### Prerequisites

If you’re brand new to web development with Django, I recommend first reading my previous
book Django for Beginners3 . The first several chapters are available for free online and cover
proper set up, a Hello World app, and a Pages app. The full-length version goes deeper and
covers a Blog website with forms and user accounts as well as a production-ready Newspaper
site that features a custom user model, complete user authentication flow, emails, permissions,
deployment, environment variables, and more.

This background in traditional Django is important since Django REST Framework deliberately
mimics many Django conventions.

It is also recommended that readers have a basic knowledge of Python itself. Truly mastering
Python takes years, but with just a little bit of knowledge you can dive right in and start building
things.

### Why APIs

Django was first released in 2005 and at the time most websites consisted of one large monolithic
codebase. The “back-end” consisted of database models, URLs, and views which interacted with
the “front-end” templates of HTML, CSS, and JavaScript that controlled the presentational layout
of each web page.

However in recent years an “API-first” approach has emerged as arguably the dominant paradigm
in web development. This approach involves formally separating the back-end from the front-
end. It means Django becomes a powerful database and API instead of just a website framework.
Today Django is arguably used more often as just a back-end API rather than a full monolithic
website solution at large companies!

An obvious question at this point is, “Why bother?” Traditional Django works quite well on its own
and transforming a Django site into a web API seems like a lot of extra work. Plus, as a developer,
you then have to write a dedicated front-end in another programming language.
This approach of dividing services into different components, by the way, is broadly known as
Service-oriented architecture4 .

It turns out however that there are multiple advantages to separating the front-end from
the back-end. First, it is arguably much more “future-proof” because a back-end API can be
consumed by any JavaScript front-end. Given the rapid rate of change in front-end libraries–
React5 was only released in 2013 and Vue6 in 2014!–this is highly valuable. When the current
front-end frameworks are eventually replaced by even newer ones in the years to come, the
back-end API can remain the same. No major rewrite is required.

Second, an API can support multiple front-ends written in different languages and frameworks.
Consider that JavaScript is used for web front-ends, while Android apps require the Java programming language, and iOS apps need the Swift programming language. With a traditional
monolithic approach, a Django website cannot support these various front-ends. But with an
internal API, all three can communicate with the same underlying database back-end!

Third, an API-first approach can be used both internally and externally. When I worked at
Quizlet7 back in 2010, we did not have the resources to develop our own iOS or Android apps.
But we did have an external API available that more than 30 developers used to create their own
flashcard apps powered by the Quizlet database. Several of these apps were downloaded over
a million times, enriching the developers and increasing the reach of Quizlet at the same time.
Quizlet is now a top 20 website in the U.S. during the school year.

The major downside to an API-first approach is that it requires more configuration than a
traditional Django application. However as we will see in this book, the fantastic Django REST
Framework library removes much of this complexity.

### Django REST Framework

There are hundreds and hundreds of third-party apps available that add further functionality to
Django. You can see a complete, searchable list over at Django Packages8 , as well as a curated
list in the awesome-django repo9 . However, among all third-party applications, Django REST
Framework is arguably the killer app for Django. It is mature, full of features, customizable,
testable, and extremely well-documented. It also purposefully mimics many of Django’s tra-
ditional conventions, which makes learning it much faster. And it is written in the Python
programming language, a wonderful, popular, and accessible language.

If you already know Django, then learning Django REST Framework is a logical next step. With a
minimal amount of code, it can transform any existing Django application into a web API.

### Why this book

I wrote this book because there is a distinct lack of good resources available for developers
new to Django REST Framework. The assumption seems to be that everyone already knows all
about APIs, HTTP, REST, and the like. My own journey in learning how to build web APIs was
frustrating… and I already knew Django well enough to write a book on it!

This book is the guide I wish existed when starting out with Django REST Framework.
Chapter 1 begins with a brief introduction to web APIs and the HTTP protocol. In Chapter 2 we
review the differences between traditional Django and Django REST Framework by building out
a Library book website and then adding an API to it. Then in Chapters 3-4 we build a Todo API
and connect it to a React front-end. The same process can be used to connect any dedicated
front-end (web, iOS, Android, desktop, or other) to a web API back-end.

In Chapters 5-9 we build out a production-ready Blog API which includes full CRUD functionality.
We also cover in-depth permissions, user authentication, viewsets, routers, documentation, and
more.

Complete source code for all chapters can be found online on Github10 .

### Conclusion

Django and Django REST Framework is a powerful and accessible way to build web APIs. By the
end of this book you will be able to build your own web APIs from scratch properly using modern
best practices. And you’ll be able to extend any existing Django website into a web API with a
minimal amount of code.

</div>