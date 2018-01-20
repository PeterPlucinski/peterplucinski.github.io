---
layout: post
title: Customizing Laravel's default authentication
tags: ['php', 'laravel']
published: true
---

Laravel’s default authentication scaffolding is very handy but sometimes you will need to modify it to suit your owen needs. This article addresses a few common cases where you may need to modify the default authentication provided to suit your requirements.

At the time of writing Laravel is at version 5.5 so if you are using a different version things may change a little.

![Levers](/img/lever.jpg "Levers")

## Stronger password requirements

The default and only password requirement in Laravel is a minimum length of 6 characters. Lets say we wanted the following password requirements:
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

