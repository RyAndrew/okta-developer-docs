Use the [Vue Router](https://router.vuejs.org/) and the [Okta Vue SDK](https://github.com/okta/okta-vue) libraries to simplify your component and route definitions.  Some routes require authentication in order to render and other don't. The following are some basic routes you need to configure for your app:

* A [default page](#default-page-route) to handle the landing page of the app.
* A [login route](#login-route) to show your app sign-in page.
* A [callback route](#callback-route) to parse tokens after a redirect from Okta.
* A [protected route](#protected-route) for authenticated users to access protected content.

#### Default page route

To create the default `/index` page, update the `src/App.vue` file to provide links to relevant locations in your app. You need to provide a link to your sign-in page, a link to sign-out of your authenticated session, and links to authenticated pages by using the `authState` property (see [`authStateManager` in Auth JS SDK](https://github.com/okta/okta-auth-js#authstatemanager)).

You can use the following condition to hide or show specific elements:

`v-if="authState && !authState.isAuthenticated"`

See the following `src/App.vue` file example:

```html
<template>
  <div id="app">
    <h1>Okta Identity Engine Vue.js and Auth JS example</h1>
      <div id="nav">
        <router-link to="/about">About</router-link> |
        <router-link to="/dashboard" v-if="authState && authState.isAuthenticated" >Dashboard</router-link>
        <span v-if="authState && authState.isAuthenticated"> | </span>
        <a v-if="authState && authState.isAuthenticated" v-on:click="logout()"> Sign out</a>
        <router-link v-if="authState && !authState.isAuthenticated" to="/login">Sign in</router-link>
      </div>
    <template v-if="$route.matched.length">
      <router-view></router-view>
    </template>
    <template v-else>
      <h2>Welcome!</h2>
      <p>This is an embedded Okta auth SPA app with Vue.js and Auth JS SDK</p>
      <p>You are {{ (authState && authState.isAuthenticated) ? '' : 'not' }} signed in.</p>
    </template>
  </div>
</template>

<script>
export default {
  name: 'app',
  methods: {
      async logout () {
        const publicPath = this.$route.href.replace(new RegExp(this.$route.fullPath + '$'), '');
        await this.$auth.signOut({ postLogoutRedirectUri: `${window.location.origin}${publicPath}` })
      }
    }
}
</script>

...

```

#### Login route

This route directs the user to your app sign-in page. See the `Login` route component, specified in the `src/components/Login.vue` file, from [Create a sign-in component](#create-a-sign-in-component).

#### Callback route

The [Okta Vue SDK](https://github.com/okta/okta-vue) provides the [LoginCallback](https://github.com/okta/okta-vue#use-the-logincallback-component) component for the callback route. It handles token parsing, token storage, and redirecting to a protected page after a user is authenticated.

See how the callback route component is called from the route definition file (`src/router/index.js`) in the [Connect the routes](#connect-the-routes) section.

#### Protected route

Create a protected route that is only available to users with a valid `accessToken`:

* You can provide access to a protected route when the `requiresAuth` metadata is added in the [route definition](#connect-the-routes).
* You can use the `authState.isAuthenticated` property in the [default route](#default-page-route) to determine if you need to show a protected element.

> **Note**: The `authState.isAuthenticated` property is true if both `accessToken` and `idToken` are valid.

In this protected `/dashboard` component example, the `src/components/Dashboard.vue` file is created to show basic information from the ID token. ID token information can be retrieved by using the [`$auth`](https://github.com/okta/okta-vue#auth) instance from the Okta Auth JS SDK and calling the [`$auth.getUser()`](https://github.com/okta/okta-auth-js#getuser) function.

```html
<template>
  <div>
    <h1>Dashboard</h1>
    <h3>Success</h3>
       <p>
       You have signed in to an embedded Okta auth SPA app with Vue.js and Auth JS SDK.
       </p>
    <h3>User Profile</h3>
    <br><br>
    <table>
      <thead>
        <tr>
          <th>Claim</th>
          <th>Value</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(claim, index) in claims" :key="index">
          <td>{{claim.claim}}</td>
          <td :id="'claim-' + claim.claim">{{claim.value}}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
export default {
  name: 'Dashboard',
  data () {
    return {
      claims: []
    }
  },
  async created () {
    this.claims = await Object.entries(await this.$auth.getUser()).map(entry => ({ claim: entry[0], value: entry[1] }))
  }
}
</script>

...

```

> **Note**: You can extend the set of claims by modifying the [scope settings in your Okta configuration](#set-up-the-okta-configuration-settings) to retrieve custom information about the user. This includes `locale`, `address`, `groups`, and [more](/docs/reference/api/oidc/#scope-values).


This snippet from the `App.vue` file provides access to the `/dashboard` component only if `authState.isAuthenticated` is true:

```html
<router-link to="/dashboard" v-if="authState && authState.isAuthenticated" >Dashboard</router-link>
```

This snippet from the `src/router/index.js` router file provides access to the `/dashboard` component only if `requiresAuth` metadata is true:

```js
{ path: '/dashboard', name: 'Dashboard', component: Dashboard, meta: { requiresAuth: true } },
```

### Connect the routes

This route definition example uses the [Vue Router](https://router.vuejs.org/) and the [Okta Vue SDK](https://github.com/okta/okta-vue) in the `src/router/index.js` file. Notice that the `requiresAuth` metadata needs to be true for protected routes.

```js
import { createRouter, createWebHistory } from 'vue-router'
import { LoginCallback } from '@okta/okta-vue'
import { navigationGuard } from '@okta/okta-vue'
import About from '@/components/About.vue'
import Dashboard from '@/components/Dashboard.vue'
import Login from '@/components/Login.vue'

const router = createRouter({
  history: createWebHistory(__dirname),
  routes: [
    { path: '/', name: 'Home', component: About },
    { path: '/about', name: 'About', component: About },
    { path: '/dashboard', name: 'Dashboard', component: Dashboard, meta: { requiresAuth: true } },
    { path: '/login', component: Login },
    { path: '/login/callback', component: LoginCallback }
  ]
})

router.beforeEach(navigationGuard)

export default router;

```

> **Note** The [Okta Vue SDK](https://github.com/okta/okta-vue#readme) provides navigation guard logic to circumvent navigational guard mixins issue in `vue-router-next`.

### Start your app

To run and test your app, execute:

```bash
npm run serve
```

Open a browser and navigate to your app URL. Try to sign in to your app with an [existing user from your Okta org](/docs/guides/quickstart/cli/main/#add-a-user-using-the-admin-console).
