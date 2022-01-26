### Set up the Okta configuration settings

Initialize the Sign-In Widget for your sign-in page by setting the required [configuration settings](#okta-org-app-integration-configuration-settings) for your app:
* `clientId`: Your client ID &mdash; `${yourClientId}`
* ` issuer`: The authorization server in your Okta org &mdash; `${yourIssuer}`
* `useInteractionCodeFlow`: Set this option to `true` to enable Identity Engine features in the embedded SIW.
* `pkce`: Set this option to `true` to enable Authentication Code with PKCE flow.
* `scopes`: Set the OAuth 2.0 scopes that your app requires.
* `redirectUri`: Set your callback redirect URI. This value must be configured in your Okta app **Sign-in redirect URIs** and **Trusted Origins** lists.

You can create a `config.js` file to define your configuration settings. For example:

```js
export default {
  oidc: {
    clientId: '${yourClientId}',
    issuer: '${yourIssuer}',
    redirectUri: '${yourLocalAppDomain}/login/callback',
    scopes: ['openid', 'profile', 'email'],
    pkce: true,
    useInteractionCodeFlow: true
  }
}
```

> **Note:** The Sign-In Widget `baseUrl` setting is not a required for OIDC applications as of SIW version 5.15.0. `['openid', 'profile', 'email']` are commonly used scopes. See [Scopes](/docs/reference/api/oidc/#scopes) for details of additional supported scopes.

### Instantiate OktaAuth

Instantiate `OktaAuth`, with the settings in [Set up the Okta configuration settings](#set-up-the-okta-configuration-settings), in your `main.js` file:

```js
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import { OktaAuth } from '@okta/okta-auth-js'
import OktaVue from '@okta/okta-vue'

import sampleConfig from '@/config'

const oktaAuth = new OktaAuth(sampleConfig.oidc)

createApp(App)
  .use(router)
  .use(OktaVue, {
    oktaAuth,
    onAuthRequired: () => {
      router.push('/login')
    },
    onAuthResume: () => {
      router.push('/login')
    },
  })
  .mount('#app')
```

### Create a SIW container

To render the SIW in Vue.js, you must create a wrapper that allows Okta to treat it as a Vue component. For example, create a `src/components/Login.vue` file with the following content:

```html
<template>
  <div class="login">
    <div id="okta-signin-container"></div>
  </div>
</template>

<script>
import OktaSignIn from '@okta/okta-signin-widget'
import '@okta/okta-signin-widget/dist/css/okta-sign-in.min.css'

import sampleConfig from '../config'

export default {
  name: 'Login',
  mounted: function () {
    this.$nextTick(function () {
      const { issuer, clientId, redirectUri, scopes, useInteractionCodeFlow } = sampleConfig.oidc
      this.widget = new OktaSignIn({
        clientId,
        redirectUri,
        logo: require('@/assets/logo.png'),
        i18n: {
          en: {
            'primaryauth.title': 'Sign in to my Okta SIW Vue.js app'
          }
        },
        authParams: {
          issuer,
          scopes,
        },
        useInteractionCodeFlow
      })

      const originalUri = this.$auth.getOriginalUri();
      if (!originalUri) {
        this.$auth.setOriginalUri('/');
      }

      this.widget.showSignInToGetTokens({
        el: '#okta-signin-container',
        scopes
      }).then(tokens => {
        this.$auth.handleLoginRedirect(tokens)
      }).catch(err => {
        throw err
      })

    })
  },
  unmounted () {
    // Remove the widget from the DOM on path change
    this.widget.remove()
  }
}
</script>
```