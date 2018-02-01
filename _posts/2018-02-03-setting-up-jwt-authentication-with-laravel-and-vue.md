---
layout: post
title: Setting up JWT authentication with Laravel and Vue JS
tags: ['php', 'laravel', 'vuejs']
published: false
---

In this tutorial I’ll cover how to setup JSON Web Token authentication using Laravel and Vue JS. It has two parts: setting up Laravel to use JWT and then authentication using Vue JS.

The Laravel documentation covers the Laravel Passport feature which is an OAuth2 implementation and a little different.

## Setting up JSON Web Tokens authentication with Laravel

We’ll be using the popular library [tymondesigns/jwt-auth](https://github.com/tymondesigns/jwt-auth) by Sean Tymon.

I’ve reproduced the steps from the documentation by Sean with a couple of minor tweaks I made to get around issues I ran into with the installation and to hopefully help readers with a couple of minor gotchas. The documentation can be found here:
* [Laravel Installation](https://jwt-auth.readthedocs.io/en/develop/laravel-installation/)
* [Quickstart](https://jwt-auth.readthedocs.io/en/develop/quick-start/)
