---
layout: post
title: Setting up JWT authentication with Laravel and Vue JS - Part 2
tags: ['php', 'laravel', 'vuejs']
published: false
---

This second and final part will focus on setting up Vue JS. We'll be creating a login page, dashboard page which requires authentication, a dummy api backend route to serve data, vue router, vuex store, a guard for routes which require authentication and a logout component.

## Setting up the Vue router

Run `npm install vue-router` to install.

Under `/resources/assets/js/` create a `store.js` file:

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

## Creating the parent App component and setting up root instance of Vue JS

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



