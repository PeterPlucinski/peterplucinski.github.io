---
layout: post
title: A layered architecture approach to Laravel applications
tags: ['php', 'laravel', 'architecture', 'mvc']
---

## A layered architecture approach to Laravel applications

The default place to "put things" in Laravel a lot of the time is the controller. I've seen code bases where controllers are very "fat" - containing business logic, validation, knowledge of data layer, etc. I've also been guilty of this myself and have searched for better approches. I wanted to share this great post by Elvira Sheina, a developer on the Toptal platform, as it approaches this topic with the idea of a layerd architecture.

https://www.toptal.com/php/maintain-slim-php-mvc-frameworks-with-a-layered-structure

A brief summary of the main points made by Elvira:

- Why Active Record implementations like Laravel's Eloquent violate the Single Responsibility Principle of SOLID
- Keeping controllers thin - a controller should only accept a request and return a response. It shouldn't contain buisness logic or data layer knowledge
- Using a service layer to hold business logic
- Using repositories to hold database layer logic
- Using the decorator pattern to hold view related logic (read the post for a good explanation)

I plan on experminentig with this approach in future projects


