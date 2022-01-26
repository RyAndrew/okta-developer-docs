You can run the [sample embedded SIW Vue.js app](https://github.com/okta/samples-js-vue/tree/master/custom-login) to quickly view a simple working Vue.js app with the Sign-In Widget.

1. Ensure that you [create an app integration in Okta](#create-an-okta-app-integration) for your sample app.
2. Download the sample app: `git clone https://github.com/okta/samples-js-vue.git`
3. Go to the `custom-login` directory: `cd samples-js-vue/custom-login`
4. Install the app and its dependencies: `npm install`
5. Set the environment variables with your [Okta org app integration configuration settings](#okta-org-app-integration-configuration-settings):

  ```bash
  export ISSUER=https://${yourOktaDomain}/oauth2/default
  export CLIENT_ID=${yourAppClientId}
  export USE_INTERACTION_CODE=true
  ```

6. Run the app: `npm start`

7. Open a browser window and navigate to the app's home page: http://localhost:8080. Try to sign in to the SIW with an [existing user from your Okta org](/docs/guides/quickstart/cli/main/#add-a-user-using-the-admin-console).