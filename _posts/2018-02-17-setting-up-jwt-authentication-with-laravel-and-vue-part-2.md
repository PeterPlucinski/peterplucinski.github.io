---
layout: post
title: Setting up JWT authentication with Laravel and Vue JS - Part 2
tags: ['php', 'laravel', 'vuejs']
published: true
---

This second and final part will focus on setting up authentication with Vue JS. We'll be creating a login page, dashboard page which requires authentication, the Vue router, the Vuex store, a guard for routes which require authentication and a logout component.

(Part 1)[http://blog.peterplucinski.com/setting-up-jwt-authentication-with-laravel-and-vue-part-1/)

(Source code)[https://github.com/PeterPlucinski/laravel-vue-jwt]

## The Vue router

Run `npm install vue-router` to install the Vue router.

Under `/resources/assets/js/` we'll create a `routes.js` file:

```
import Vue from 'vue'
import VueRouter from 'vue-router'

Vue.use(VueRouter)

import DashboardComponent from './components/DashboardComponent'
import LoginComponent from './components/LoginComponent'
import LogoutComponent from './components/LogoutComponent'

const routes = [
    {
        path: '/',
        redirect: { name: 'login' }
    },
    {
        path: '/dashboard',
        name: 'dashboard',
        component: DashboardComponent
    },
    {
        path: '/login',
        name: 'login',
        component: LoginComponent
    },
    {
        path: '/logout',
        name: 'logout',
        component: LogoutComponent
    }
]

const router = new VueRouter({
    routes
})

export default router
```

Here we are setting up four routes which will point to corresponding components. I have included the import statements for our components and we'll create these in a second. The root path `/` simply redirects to the login page.

## App component and the root instance of Vue JS

Next we want to setup a parent component for our app. This will hold all of our child components.

Create the file `/resources/assets/js/components/AppComponent.vue`:

```
<template>
    <router-view></router-view>
</template>

<script>
    export default {}
</script>
```

This will simply contain the output of our Vue router.

The app component is used inside the `app.blade.php` Laravel template:

```
<div id="app">
    <app-component></app-component>
</div>
```

We'll also need to modify our `app.js` file to import our router and main App component into the Vue JS instance:

```
import router from './routes.js';
import AppComponent from './components/AppComponent'

const app = new Vue({
    components: { AppComponent },
    router
}).$mount('#app')
```

## The Vuex store

Install vuex by running `npm install vuex --save-dev`.

Create a `store.js` file inside `/resources/assets/js/`. 

In larger Vue applications, which can contain vuex modules, the convention tends to be to use `store/index.js` but I'm keeping things fairly simple here.

```
import Vue from 'vue'
import Vuex from 'vuex'

Vue.use(Vuex)

export default new Vuex.Store({
    state: {
        isLoggedIn: !!localStorage.getItem('token')
    },
    mutations: {
        loginUser (state) {
            state.isLoggedIn = true
        },
        logoutUser (state) {
            state.isLoggedIn = false
        },
    }
})
```

Our store will allow us to login and logout our user as well as keep track of whether the user is logged in even if the page is refresehd.

One important point here is that the token is stored in browser local storage. The other option would be to use a cookie for this. There are pros and cons to both approaches and you may want to consider their implications before using this code in a production application.

The `isLoggendIn` property will get its initial value based on whether we have a token stored in local storage. This is so that when a user refreshes the page they will still remain logged in. The `!!` (not, not) operator simply coerces the call to `getItem()` to a Boolean.

## Login component

Next, we'll create our login component which shows the login page and allows us to submit login information to our API. The API will then send back a JSON web token if login is successfull.

Create a `LoginComponent.vue` inside `/resources/assets/js/components/`:

```
<template>
    <div class="text-center form-wrapper">

        <form class="form-signin" v-on:submit.prevent="submitLogin">
            <img class="mb-4" src="https://getbootstrap.com/assets/brand/bootstrap-solid.svg" alt="" width="72" height="72">
            <h1 class="h3 mb-3 font-weight-normal">Please sign in</h1>

            <label for="inputEmail" class="sr-only">Email address</label>
            <input type="email" id="inputEmail" class="form-control" placeholder="Email address" required autofocus v-model="email">

            <label for="inputPassword" class="sr-only">Password</label>
            <input type="password" id="inputPassword" class="form-control" placeholder="Password" required v-model="password">

            <button class="btn btn-lg btn-primary btn-block" type="submit">Sign in</button>
        </form>

    </div>
</template>

<script>
    import store from '../store'
    export default {
        data() {
            return {
                email: '',
                password: '',
                loginError: false,
            }
        },
        methods: {
            submitLogin() {
                this.loginError = false;
                axios.post('/api/auth/login', {
                    email: this.email,
                    password: this.password
                }).then(response => {
                    // login user, store the token and redirect to dashboard
                    store.commit('loginUser')
                    localStorage.setItem('token', response.data.access_token)
                    this.$router.push({ name: 'dashboard' })
                }).catch(error => {
                    this.loginError = true
                });
            }
        }
    }
</script>

<style scoped>
    .form-wrapper {
        min-height: 100%;
        min-height: 100vh;
        display: flex;
        align-items: center;
    }
    .form-signin {
        width: 100%;
        max-width: 330px;
        padding: 15px;
        margin: 0 auto;
    }
    .form-signin .form-control {
        position: relative;
        box-sizing: border-box;
        height: auto;
        padding: 10px;
        font-size: 16px;
    }
    .form-signin .form-control:focus {
        z-index: 2;
    }
    .form-signin input[type="email"] {
        margin-bottom: -1px;
        border-bottom-right-radius: 0;
        border-bottom-left-radius: 0;
    }
    .form-signin input[type="password"] {
        margin-bottom: 10px;
        border-top-left-radius: 0;
        border-top-right-radius: 0;
    }
</style>
```

A few noteworthy points. The `submitLogin()` method is called when a user attempts to login. This method:
* Creates a post request using axios to our backend API authentication route `/api/auth/login`
* On success, it "commits" a vuex store mutation `loginUser()` which sets the `isLoggedIn` property to `true`
* Saves the token returned from our API backend in browser local storage
* Uses the Vue router to redirect to a protected dashboard page

I've included the styling here. It's borrowed from the Bootstrap login example.

## Dashboard component

The dashboard component will be our "internal" page. A user must login before accessing this page. We'll look at protecting this page a little later.

Create a `DashboardComponent.vue` file inside `/resources/assets/js/components/`:

```
<template>
    <div class="container">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <h1>Dashboard</h1>

                <p>
                    <router-link :to="{ name: 'dashboard' }">Dashboard</router-link> |
                    <router-link :to="{ name: 'login' }">Login</router-link> |
                    <router-link :to="{ name: 'logout' }">Logout</router-link>
                </p>

                <div class="panel panel-default">
                    <div class="panel-heading">Dashboard</div>
                    <div class="panel-body">
                        <p>Data: "{{ data }}"</p>
                    </div>
                </div>

            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                data: 'nothing'
            }
        },
        mounted() {
            axios.get('/api/dashboard', {
                headers: {
                    Authorization: 'Bearer ' + localStorage.getItem('token')
                }
            })
            .then(response => {
                this.data = response.data.data
            }).catch(error => {

            })
        }
    }
</script>
```

This component includes router links for our routes. The most notable part is the `mounted()` method where we make a call to the backend using our token from local storage.

In Laravel, we need to create the backend dashboard route inside our `api.php` routes file:

```
Route::middleware('auth:api')->group(function () {
    Route::get('dashboard', function () {
        return response()->json(['data' => 'Test Data']);
    });
});
```

This route is protected by the `auth:api` middleware which will check for a valid token before returing data. A 401 (Unauthorized) response will be returned if there is no valid token with the API call.

## Logout component

Create a `LogoutComponent.vue` file inside `/resources/assets/js/components/`:

```
<template>

</template>

<script>
    import store from '../store'
    export default {
        mounted () {
            localStorage.removeItem('token')
            store.commit('logoutUser')
            this.$router.push({ name: 'login' })
        }
    }
</script>
```

This should be fairly self explanatory. On logout, we remove the token, set `isLoggedIn` to `false` and redirect the user to the login page.

## Guarding internal pages

The final step is to setup a navigation guard for pages which require authentication using Vue JS. To do this we'll modify the `routes.js` file.

Firstly, we need to add the import statement for our store:

```
import store from './store'
```

Next, we'll add a meta property to our dashboard route. We can add the same property to any route requiring authentication.

```
{
    path: '/dashboard',
    name: 'dashboard',
    component: DashboardComponent,
    meta: { requiresAuth: true }  // add this
},
```


Finally, to create the navigation guard we'll be adding a `beforeEach()` method to our router:

```
router.beforeEach((to, from, next) => {

    // check if the route requires authentication and user is not logged in
    if (to.matched.some(route => route.meta.requiresAuth) && !store.state.isLoggedIn) {
        // redirect to login page
        next({ name: 'login' })
        return
    }

    // if logged in redirect to dashboard
    if(to.path === '/login' && store.state.isLoggedIn) {
        next({ name: 'dashboard' })
        return
    }

    next()
})
```

This method checks every route for the `requiresAuth:true` meta property. If a user is not logged in, they will be redirected to the login page.

If a user is already logged in and visits the login page, they will be redirected to the dashboard page.

See the [navigation guard documentation](https://router.vuejs.org/en/advanced/navigation-guards.html) for an explanation of the `to`, `next` and `from` arguments.

That concludes this tutorial. We now have a working Vue SPA with JSON web token authentication.

I hope you have found this guide useful. Any comments or questions are welcome.
