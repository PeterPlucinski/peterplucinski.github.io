---
layout: post
title: A layered architecture approach to Laravel applications
tags: ['php', 'laravel', 'architecture', 'mvc']
---

The default place to "put things" in a Laravel project a lot of the time is the controller. I've seen and been guilty of creating "fat" controllers, which contain business logic, validation, knowledge of the data layer and relationships, etc. I've been testing different approaches and I wanted to share this great post by Elvira Sheina, a PHP developer on the Toptal platform.

[Maintain Slim PHP MVC Frameworks with a Layered Structure](https://www.toptal.com/php/maintain-slim-php-mvc-frameworks-with-a-layered-structure)

A brief summary of the main points:

- Why Active Record implementations like Laravel's Eloquent violate the Single Responsibility Principle of SOLID
- Keeping controllers thin - a controller should only accept a request and return a response. It shouldn't contain buisness logic or data layer knowledge
- Using a service layer to hold business logic
- Using repositories to hold database layer logic
- Using the decorator pattern to hold view related logic (read the post for a good explanation)

I plan on experimentig with this approach in future projects and I'll likely share my findings.


