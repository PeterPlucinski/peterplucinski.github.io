---
layout: post
title: Why use a framework like Laravel
tags: ['php', 'laravel', 'frameworks']
published: true
---

![Structure](/img/bridge.jpg "Strucutre")

Frameworks like Laravel are very popular and have been around for a while in the PHP community but they haven't been embraced by everyone and their benefits aren't always apparent. While there may be a few cases where using a framework doesn’t make sense, for the most part, there are significant benefits in using a framework to build a web application.

I’ve been in a few discussions about frameworks and decided to write a blog post to present my opinion in one place.

I’ll explain why you would want to consider a framework like Laravel and how frameworks can help developers write clean and maintainable code, while also potentially saving time.

## A well defined structure

Modern PHP frameworks follow the MVC pattern. They enforce a separation between Model, View and Controller code. MVC is a well known and widely used architecture. It ensures that the application follows a well defined architecture from the beginning. As soon as the developer goes to work and begins to write the first few parts of an application, they are already following MVC as the files are separated by default for each concern in a framework. Models, views and controllers have their own folders and files. It’s a little more difficult to mix these concerns when using a framework. On the other hand, it is very easy to mix concerns when writing an application in PHP. There is a lot of legacy PHP code out there and a majority of the time design patterns like MVC aren’t followed. MVC promotes a cleaner, more readable and more maintainable code structure.

Frameworks follow a predefined directory structure. In addition to models, views and controllers, most frameworks have a set place for configuration files, public facing assets such as JavaScript and CSS, as well as other framework features such as for example events, middleware and database migrations. This means that another developer familiar with the framework will be able to get up to speed with a code base much quicker as there is already an agreed upon directory structure. This is unlike applications written in vanilla PHP, where there is no single, well defined structure that everyone is familiar with.

It’s important to note that Laravel is quite flexible in where it will allow you to put things while still using the basic directory structure. For example, it’s very easy to create additional folders for things like Interfaces or Repositories. With the use of namespaces, it’s also possible to group Models, Interfaces and Repositories together in one directory when they relate to a single component.

## Core components included

Frameworks like Laravel are said to be “batteries included”. They provide all the most common building blocks used in web applications. For example, most frameworks include a router, an ORM, form validation functionality and helper methods for dealing with file uploads or HTTP requests in a clean manner. Many of Laravel’s underlying components come from the well regarded and battle tested Symfony framework. The HTTP Request component which represents a request entering the application is an example of this.

What this translates to is that there is no need to reinvent the wheel every time you build a new web application. You can make use of included components instead of writing the functionality they provide from scratch.

Core components also abstract the programmer away from having to deal with low level details. It’s a bit like writing something in an interpreted language like PHP versus having to deal with all the underlying details, such as memory management when writing a program in a lower level language such as C. For example, when using the Request and Response objects in Laravel you don’t have to worry about the nuances of the underlying HTTP protocol. This work is done for you already.

These components are open source, well tested and follow current best practices. You could decide to build your own components and use these every time you build a new web application. However, if you use components provided by frameworks like Laravel you can be sure that they are solid. It’s a good idea to build on something that is open source and has had many eyeballs on it.

The ability to build your application on well tested, open source components is one of the biggest benefits of using a framework like Laravel.

## Helpful features

In addition to the core components, frameworks also contain many helpful features. Laravel contains a plethora of inbuilt helper functions to make your life as a developer easier. Complex functionality has been packaged up into easy to use statements. Here are some of the features of Laravel at a glance.

Form Validation. Laravel includes a form validation component complete with error messages and a large range of common validation rules to simplify the process of validating user input.

Pagination. Laravel provides easy to use database pagination out of the box. Pagination links can easily be added using a helper and all of the pagination logic is handled by Laravel.

Events and Listeners. Laravel provides a clean API for firing events and creating listeners. A common use case for these is email notifications.

Testing. Integration with PHPUnit is included out of the box. Laravel also comes with a suite of helper functions to allow you to test the functionality of your web application using fluent, expressive syntax.

Collections. Laravel provides a wrapper for working with database results and arrays of data, which contains further helpful methods such as map and reduce. These methods can be chained together and make it easy accomplish many data manipulation tasks.

Commands. Laravel includes a command line interface called “artisan”. You can also build your own custom commands. Laravel makes building command line tools a breeze.

Model Factories. Model factories allow easy creation of Models in your application using dummy data. Model factories can be called using artisan or inside tests to populate the database.

Queues. Queues can be used to defer the processing of a time consuming task, such as sending a batch of email notifications, till a later time. Laravel includes drivers for popular que backends such as Redis and Beanstalkd.

Middleware. Middleware is a mechanism for filtering HTTP requests entering your application. Laravel provides authentication middleware to ensure that only authenticated users can reach certain parts of your application. You can easily create your own middleware in Laravel.

Task Scheduling. Laravel allows you to define scheduled tasks to be run at set intervals using a clean API. A single cron entry is then needed to call Laravel’s task scheduler.

Laravel Mix. Laravel provides a clean API for compiling and minifying CSS, SASS, LESS and JavaScript.

These are just a few examples. There are many more helpful features included with Laravel and I’d encourage anyone interested to check out the documentation. By using these features developers can save time as there is less code to write. It also means there is less code to debug, less code to maintain and less code for a new developer to get familiar with when they are getting started with an existing project.

## Routing and pretty URLs

Most modern frameworks include a router which maps specific HTTP requests to different parts of the code, based on the URI. A single route will usually map to a controller method. The controller’s job is to take a HTTP request and return a response.

The router adds a degree of separation between the URL and the files used to produce a certain page. There is no longer a one-to-one correlation between the URL and the PHP script doing the work for that page. You can also be a lot more flexible in how you structure routes, because you are no longer limited to using directories to create a URL structure.

By employing this flexibility you can create URLs which are clean and friendly to both users and search engines. There is also the advantage that when the underlying functionality used to serve a page needs to change, the URL can remain unchanged.

## ORM

Full featured frameworks like Laravel (as opposed to micro frameworks) include an ORM for working with the database. In an ORM, a database table is represented by a Model. A table row is represented by an object. Table columns are accessed using properties of the object. 
Once the mappings and objects are created, the objects completely manage the application’s data access needs. There is no need to write low level code to access the database. The ORM provides a host of functions allowing the developer to focus more on the business logic of the application rather than repetitive CRUD (Create Read Update Delete) logic present in most data driven web applications.

The code used to access database records is PHP. Therefore your code base is no longer tied to specific SQL syntax. This makes it easier to switch between different database vendors. Laravel includes a number of different database drivers out of the box - MySQL, SQLite and PostgreSQL. 

Laravel’s ORM, Eloquent, provides a clean and expressive syntax for working with database records. Eloquent also provides the ability to create complex relationships between models. You can still use raw SQL in a Laravel project if the need ever arises.

## Templating

Laravel views use a powerful templating engine called Blade. The main benefit of using templating over standard PHP files is the use of template inheritance and sections.

Using Blade you can define a master template which other views can then inherit from. An entire hierarchy can be created in this way. Using templating is simpler and cleaner than using PHP itself for templating.

Blade also automatically escapes HTML output so there is no need to do this manually as you would need to do with ordinary PHP. It also offers useful filters. For example, you can tell Blade that you would like to output prices as a certain currency with two decimal places by using the currency filter.

It’s worth noting that unlike other templating engines Blade does not prevent you from using plain PHP inside your views if you need to do something more complex.

## Developer productivity

Frameworks improve developer productivity. You will be able to get something up and running much quicker if you are using a framework. This is because there is less code to write. As mentioned earlier, frameworks include core components and helper features. If you wanted to start from scratch without using a framework you would need to write a lot of the functionality yourself. For example, a developer will no longer need to spend time implementing authentication as it’s provided by the framework.

The time saved in low value added tasks can then be used in other ways. For example more time can be spent on designing the application architecture to ensure that future business requirements can be implemented more efficiently. Or more time can be spent on testing.

The time savings are especially important if you are a startup creating an MVP and will likely need to iterate on the product. It’s also possible that you want to create a web application as quickly as possible to see how things could look and function.

## Performance tools

Most frameworks contain built in performance profiling tools as well as functionality to improve performance in your web application. 

Laravel has a package called “Debug Bar”, which can help analyse performance and assist in debugging. The Debug Bar can display the raw database queries being run to produce a certain page. It can also show the total response time for a certain HTTP request as well as memory usage.

Caching can be used to speed up your web application by storing frequently requested data where it can be accessed very quickly, such as in memory. Laravel provides support for a number of different caching methods, such as the filesystem and database, as well as drivers for two popular caching backends, Redis and Memcached.

Laravel’s ORM, Eloquent includes a feature called Eager Loading. Eager Loading can be used to load all attached (via a relationship) records using a single database query. This reduces the amount of queries that have to run and alleviates the N+1 problem. See the Laravel documentation for further explanation of eager loading.

## Security

Frameworks contain a lot of functionality to help you build secure web applications. One piece of functionality is CSRF (Cross Site Request Forgery) protection. Laravel will generate a CSRF token as a hidden field in your form and automatically verify that the token is genuine when the forms are submitted.

Most frameworks contain authentication functionality built in. Laravel includes infrastructure for logging in a user, remembering a user’s session and resetting passwords via email. 

There are a few other security features included with Laravel:

* Blade templates automatically escape output
* Laravel cookies are encrypted by default so they can’t be read by the client
* A helper method for hashing passwords

## Conclusion

There are significant benefits to using a framework like Laravel over coding applications in regular PHP. Frameworks help developers write well structured and maintainable web applications. I have focused on the Laravel framework but all of the benefits apply to other PHP frameworks like Symfony for example. They are also applicable to frameworks in other languages like for example the Python based frameworks Django and Flask.

I hope you have enjoyed the article. Comments and feedback are welcome.



