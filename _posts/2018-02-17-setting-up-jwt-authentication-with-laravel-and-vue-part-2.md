---
layout: post
title: Setting up JWT authentication with Laravel and Vue JS - Part 2
tags: ['php', 'laravel', 'vuejs']
published: true
---

This second and final part will focus on setting up Vue JS. We'll be creating a login page, dashboard page which requires authentication, a dummy api backend route to serve data, vue router, vuex store, a guard for routes which require authentication and a logout component.

## The Vue router

Run `npm install vue-router` to install.

Under `/resources/assets/js/` create a `routes.js` file:

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

## Parent App component and root instance of Vue JS

Next we want to setup a parent component for our app. This will hold child components.

Create the file `/resources/assets/js/components/AppComponent.vue`

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

Next, we'll need to modify our `app.js` file to import our router and main App component:

```
import router from './routes.js';
import AppComponent from './components/AppComponent'

const app = new Vue({
    components: { AppComponent },
    router
}).$mount('#app')
```

## Vuex store

Install vuex by running `npm install vuex --save-dev`.

Create a `store.js` file inside `/resources/assets/js/`. 

In larger Vue applications, which can contain vuex modules, the convention tends to be to use `store/index.js` but I'm keeping it failry simple here.

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

Our store will allow us to login, logout and keep track of whether the user is logged in even if the user refreshed the page. One important point here is that I'm storing the JSON web token inside browser local storage. The other option would be to use a cookie for this. There are pros and cons to both approaches and you may want to consider the implications of each before using this code in a production app.

The `isLoggendIn` property will get its initial value based on whether we have a token saved in local storage. This is so that when a user refreshes the page they can still remain logged in. The `!!` (not, not) operator simply coerces the call to `getItem()` to a Boolean.

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
* On success, it "commits" a vuex store mutation `loginUse` which sets the `isLoggedIn` property to `true`
* Saves the token returned from our API backend in browser local storage
* Uses the Vue router to redirect to a protected dashboard page

The styling for the login form is borrowed from the Bootsrap login example.

## Dashboard component

The dashboard component will be our "internal" page. A use must login before being given access. We'll look at protecting this page in a minute.

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
                        <p>Data: {{ data }}</p>
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

This component includes router links for our routes. Likely the most notable part is the `mounted()` method where we make a call to the backend using our token stored from local storage.

Create the backend dashboard route like below inside our `api.php` routes file:

```
Route::middleware('auth:api')->group(function () {
    Route::get('dashboard', function () {
        return response()->json(['data' => 'Test Data']);
    });
});
```

This route is protected by the `auth:api` middleware which will check for a valid before returing the data. A 401 (Unauthorized) response will be returned if no valid token is present with the API request.

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

This is failry self explanatory. We remove the token, set `isLoggedIn` to false and redirect to the login page.

## Guarding internal pages

The final step is to setup a navigation guard for pages which require authentication. To do this we'll modify the `routes.js` file.

Firstly, we also need to add the import statement for our store:

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


Finally, to create the guard we'll be add a `beforeEach()` method to our router:

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

This method checks every route requested for the `requiresAuth: true` meta property. If a user is not logged in they will be redirected to the login route.

If a user is already logged in and visits the login page, they will be redirected to the dashboard page.

See the [Vue router documentation](https://router.vuejs.org/en/advanced/navigation-guards.html) for an explanation of `to`, `next` and `from`.

I hope you have found this guide useful. Any comments or questions welcome.
