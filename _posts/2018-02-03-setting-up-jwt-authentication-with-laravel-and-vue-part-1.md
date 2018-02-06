---
layout: post
title: Setting up JWT authentication with Laravel and Vue JS - Part 1
tags: ['php', 'laravel', 'vuejs']
published: true
---

In this tutorial I’ll cover how to setup JSON Web Token authentication using Laravel and Vue JS. The tutorial will have two parts. The first part will cover setting up Laravel to generate JSON Web Tokens. The second part will cover authentication using Vue JS in the context of an SPA.

The approach here is different to what the Laravel documentation covers - the Laravel Passport feature which is an OAuth2 implementation.

Source code can be found [here](https://github.com/PeterPlucinski/laravel-vue-jwt).

![Actors](/img/actor.jpg "Actors")

## Setting up JSON Web Token authentication with Laravel

To generate JSON Web tokens from the Laravel backend we’ll be using the popular library [tymondesigns/jwt-auth](https://github.com/tymondesigns/jwt-auth) by Sean Tymon.

I’ve reproduced the steps from the documentation by Sean with a couple of minor tweaks I made to get around issues I ran into with the installation and to hopefully help readers with a couple of minor gotchas. The documentation can be found here:
* [Laravel Installation](https://jwt-auth.readthedocs.io/en/develop/laravel-installation/)
* [Quickstart](https://jwt-auth.readthedocs.io/en/develop/quick-start/)

### Installation

Install the library. I've included the latest version in the command. Without including the version number, on my machine composer installed version 0.5.

`composer require tymon/jwt-auth:1.0.0-rc.1`

Publish the configuration file. This creates a `config/jwt.php` config file.

`php artisan vendor:publish --provider="Tymon\JWTAuth\Providers\LaravelServiceProvider"`

Generate a secret key used to sign the tokens

`php artisan jwt:secret`

### User model

Update the User model:
* add use statement for `JWTSubject`
* add `getJWTIdentifier()` and `getJWTCustomClaims()` methods
* add `implements JWTSubject` to the class definition - this one is easy to overlook

```php
<?php

namespace App;

use Illuminate\Notifications\Notifiable;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Tymon\JWTAuth\Contracts\JWTSubject;

class User extends Authenticatable implements JWTSubject
{
    use Notifiable;

    // existing User model code

    /**
     * Get the identifier that will be stored in the subject claim of the JWT.
     *
     * @return mixed
     */
    public function getJWTIdentifier()
    {
        return $this->getKey();
    }

    /**
     * Return a key value array, containing any custom claims to be added to the JWT.
     *
     * @return array
     */
    public function getJWTCustomClaims()
    {
        return [];
    }
}
```

### auth.php configuration file

Update the `config/auth.php` file:
* set `api` as the default guard
* use `jwt` for the api driver

```
'defaults' => [
    'guard' => 'api', // updated
    'passwords' => 'users',
],

'guards' => [
    'web' => [
        'driver' => 'session',
        'provider' => 'users',
    ],

    'api' => [
        'driver' => 'jwt', // updated
        'provider' => 'users',
    ],
],
```

### API authentication routes

Add api routes for authentication.

```
Route::group([

    'middleware' => 'api',
    'prefix' => 'auth'

], function ($router) {

    Route::post('login', 'AuthController@login');
    Route::post('logout', 'AuthController@logout');
    Route::post('refresh', 'AuthController@refresh');
    Route::post('me', 'AuthController@me');

});
```

### AuthController

Create `AuthController.php` - this is the workhorse for our authentication and allows us to use Laravel's default authentication with the `jwt-auth` library.

```
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use App\Http\Controllers\Controller;

class AuthController extends Controller
{
    /**
     * Create a new AuthController instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth:api', ['except' => ['login']]);
    }

    /**
     * Get a JWT via given credentials.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function login()
    {
        $credentials = request(['email', 'password']);

        if (! $token = auth()->attempt($credentials)) {
            return response()->json(['error' => 'Unauthorized'], 401);
        }

        return $this->respondWithToken($token);
    }

    /**
     * Get the authenticated User.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function me()
    {
        return response()->json(auth()->user());
    }

    /**
     * Log the user out (Invalidate the token).
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function logout()
    {
        auth()->logout();

        return response()->json(['message' => 'Successfully logged out']);
    }

    /**
     * Refresh a token.
     *
     * @return \Illuminate\Http\JsonResponse
     */
    public function refresh()
    {
        return $this->respondWithToken(auth()->refresh());
    }

    /**
     * Get the token array structure.
     *
     * @param  string $token
     *
     * @return \Illuminate\Http\JsonResponse
     */
    protected function respondWithToken($token)
    {
        return response()->json([
            'access_token' => $token,
            'token_type' => 'bearer',
            'expires_in' => auth()->factory()->getTTL() * 60
        ]);
    }
}
```

If you haven't already you will need to run `php artisan:migrate` to create the default users table which comes with Laravel.

As a side note, I didn't need to update the `app.php` providers or aliases sections for the library to work with Laravel 5.5.

### Testing token generation

We can now test that token generation is working properly. To test I used Postman to send a POST request to `localhost:8000/api/auth/login` with a body made up of valid email and password. You will need to register a valid user before attempting this. I used Laravel's tinker to generate a new User instance in the database.

The response I got is below.

```json
{
    "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjgwMDAvYXBpL2F1dGgvbG9naW4iLCJpYXQiOjE1MTc5MDk1MTgsImV4cCI6MTUxNzkxMzExOCwibmJmIjoxNTE3OTA5NTE4LCJqdGkiOiJVbzdIVTl2cDVCWno2SFBsIiwic3ViIjoyLCJwcnYiOiI4N2UwYWYxZWY5ZmQxNTgxMmZkZWM5NzE1M2ExNGUwYjA0NzU0NmFhIn0.9j4zPng0W1fFzStNQHYj1gyuMAUT1sIFfQcKgvXCX0M",
    "token_type": "bearer",
    "expires_in": 3600
}
```

### Setting up Laravel to function as an SPA (Single Page App)

Firstly we'll need to create a blade template which contains the Vue JS application. I've create the file `/resources/views/app.blade.php` with the code below.

A couple of notable points here:
* We're adding the CSRF token for use by Vue JS as a meta tag
* The `<router-view></router-view>` will be where our Vue JS app is displayed and we'll be using Vue's routing

```
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <meta name="csrf-token" content="{{ csrf_token() }}">

    <title>Laravel</title>
    
</head>
<body>
    <div id="app">
        <router-view></router-view>
    </div>
    
    <script src="js/app.js"></script>
</body>
</html>
```

Second, we want to setup a route to point to this view inside `/routes/web.php`.

```
Route::get('/', function () {
    return view('app');
});
```

That's all for part 1. In part 2 we'll explore how to setup the Vue JS app which will consume the API and allow as to authenticate using JSON Web Tokens.
