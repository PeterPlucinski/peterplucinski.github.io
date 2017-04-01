---
layout: post
published: false
title: Layered Architecture for Your Laravel Applications
---
## Layered Architecture for Your Laravel Applications

- Why Active Record implementations like Laravel's Eloquent violate the Single Responsibility Principle of SOLID
- Keeping controllers thin - a controller should only accept a request and response. It shouldn't contain buisness logic or data layer knowledge
- Using a service layer to hold business logic
- Using repositories to hold database layer logic
- Using the decorator pattern to hold view related logic (read the post for a good explanation)


