---
layout: post
title: Customizing Laravel's default authentication
tags: ['php', 'laravel']
published: true
---

Laravel’s default authentication scaffolding is very handy but sometimes you will need to modify it to suit your own needs. This article addresses a few common cases where you may need to modify the default authentication provided to suit your requirements.

At the time of writing Laravel is at version 5.5 so if you are using a different version things may change a little.

![Levers](/img/lever.jpg "Levers")


## Stronger password requirements

The default and only password requirement in Laravel is a minimum length of 6 characters. Let's say we wanted the following password requirements:
* Minimum length of 8 characters
* At least one uppercase letter
* At least one number

To accomplish this, we need to modify the `validator` method inside the `RegisterController` class located in `/app/Http/Controllers/Auth/`. 

Specifying minimum length is very easy. For the other two requirements we have to use regex. The validation method should look like this:

```
protected function validator(array $data)
{
    return Validator::make($data, [
        'name' => 'required|string|max:255',
        'email' => 'required|string|email|max:255|unique:users',
        'password' => 'required|string|min:8|confirmed|regex:/^(?=.*\d)(?=.*[A-Z]).+$/',
    ],
    [
        'password.regex' => 'The password should contain at least one uppercase letter and at least one digit.'
    ]);
}
```

I have also added a custom message for the regex component of the password field to let the user know what is required. Without the custom error message the default message for the regex is “The password format is invalid.”

We also need to modify the `ResetPasswordController` class. The methods we want to modify are located inside the `ResetPasswords` trait so we can overwrite these methods in the `ResetPasswordController` class. The methods we want to overwrite are `rules` and `validationErrorMessages`. You can simply copy these methods from the trait into the `ResetPasswordController` class and make changes there.

This may be obvious to some but it should be mentioned that the core files of Laravel (those residing in the `/vendor` folder) should never be modified. Instead they can be overwritten where required.

The modified methods should look like below and reside in the `ResetPasswordController` class.

```
protected function rules()
{
    return [
        'token' => 'required',
        'email' => 'required|email',
        'password' => 'required|string|min:8|confirmed|regex:/^(?=.*\d)(?=.*[A-Z]).+$/',
    ];
}

protected function validationErrorMessages()
{
    return [
        'password.regex' => 'The password should contain at least one uppercase letter and at least one digit.'
    ];
}
```


## Redirect to a custom page after logging in

After logging in, the default route a user is taken to is `/home`. Let’s say that we would like to send the user to a `/dashboard` route after they are logged in.

There are actually four places where this needs to be updated. It can be easy to miss one of these - especially the last one:
* LoginController class
* ResetPasswordController class
* RegisterController class
* RedirectIfAuthenticated class - middleware

To find the instances where this needs to be changed you could just search for `/home` in the codebase.

The first three are all controllers related to authentication and these are all located in `/app/Http/Controllers/Auth/`. The last one however if middleware and its located in `/app/Http/Middleware/`.

The first 3 are straightforward:

```
// LoginController.php
// ResetPasswordController.php
// RegisterController.php

protected $redirectTo = '/dashboard';
```

Lastly we need to modify the `handle` method of the `RedirectIfAuthenticated` middleware:

```
public function handle($request, Closure $next, $guard = null)
{
    if (Auth::guard($guard)->check()) {
        return redirect('/dashboard');
    }

    return $next($request);
}
```


## Registering different user types

Let's say that we would like to be able to register as a normal user or as an "admin" user.

Firstly we'll need to modify the `register.blade.php` view file located at `/resources/views/auth/`. We'll add a select form element underneath the password confirmation to allow selection of different user types. I've kept the style same as the original register view which comes with Laravel.

```
<div class="form-group{{ $errors->has('user_type') ? ' has-error' : '' }}">
    <label for="user_type" class="col-md-4 control-label">User Type</label>
    <div class="col-md-6">
        <select name="user_type" id="user_type" class="form-control" required>
            <option value="user">User</option>
            <option value="admin">Admin</option>
        </select>
        @if ($errors->has('user_type'))
            <span class="help-block">
                <strong>{{ $errors->first('user_type') }}</strong>
            </span>
        @endif
    </div>
</div>
```

Next, we'll add a `type` column to our users migration. Migrations are located in `/database/migrations/`.

```
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->increments('id');
        $table->string('name');
        $table->string('email')->unique();
        $table->string('password');
        $table->string('type');
        $table->rememberToken();
        $table->timestamps();
    });
}
```

We also need to make sure that the `type` column is fillable by modifying the `User` model in the `/app/` folder.

```
protected $fillable = [
    'name', 'email', 'password', 'type'
];
```

We need to add a `type` field to the `create` method of the `RegisterController` located in `/app/Http/Controllers/Auth/`.

```
protected function create(array $data)
{
    return User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => bcrypt($data['password']),
        'type' => $data['user_type']
    ]);
}
```

Lastly, we'll add some validation for `user_type`. We need to modify the `validator` method of the `RegisterController`. The other validation rules remain from the earlier section on stronger passwords.

```
protected function validator(array $data)
{
    return Validator::make($data, [
        'name' => 'required|string|max:255',
        'email' => 'required|string|email|max:255|unique:users',
        'password' => 'required|string|min:8|confirmed|regex:/^(?=.*\d)(?=.*[A-Z]).+$/',
        'user_type' => 'required|in:admin,user'
    ],
    [
        'password.regex' => 'The password should contain at least one uppercase letter and at least one digit.'
    ]);
}
```

That's pretty much all that's needed to be able to register different user types.

## Disable registration

You may want to completely disable the ability to register users. This is relatively simple. We need to override the `showRegistrationForm` and `register` methods of the `RegisterUsers` trait. We can override these methods inside the `RegisterController` located in `/app/Http/Controllers/Auth/`.

```
public function showRegistrationForm()
{
    return abort(404);
}

public function register()
{
    return abort(404);
}
```

The `register` method is responsible for handling the POST request for user registration so its important the we override this method in addition to the `showRegistrationForm` method which displays the form.

I've used a 404 message but we could also redirect the user to the home page - `return redirect('/')`. If for some reason you are using the welcome page supplied with Laravel you will also want to remove the "Register" link.

That's all for this article and I hope you have found it useful. I will likely add more use cases for custom authentication. One of the big ones is API authentication using JSON Web Tokens with Vue JS for example. I'll  likely devote my next article to this topic.

Feedback and comments are welcome.
