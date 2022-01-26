### Create a new Vue.js app (optional)

If you don't have an existing Vue.js app, you can quickly create a new app by using the [Vue CLI](https://cli.vuejs.org/guide/installation.html):

```bash
npm install -g @vue/cli
vue create okta-app
```

* Manually select the following features: defaults and **Router**.
* Select **3.x** for the Vue.js version.
* Select **Y** for router history mode.

Go into your app directory to view the created files:

```bash
cd okta-app
```

### Install dependencies

Add the [Okta Sign-In Widget](/code/javascript/okta_sign-in_widget/) library into your Vue.js app. You can install it by using `npm`:

```bash
cd okta-app
npm install @okta/okta-signin-widget
```

You also need [Okta Vue SDK](https://github.com/okta/okta-vue) (`@okta/okta-vue`) for route protection and [Okta Auth JS SDK](https://github.com/okta/okta-auth-js) (`@okta/okta-auth-js`) for SIW dependencies:

```bash
npm install @okta/okta-vue
npm install @okta/okta-auth-js
```