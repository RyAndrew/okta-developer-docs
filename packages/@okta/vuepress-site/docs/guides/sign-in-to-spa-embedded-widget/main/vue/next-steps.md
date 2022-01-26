After successfully authenticating with Okta, the app obtains the user's `id_token`, which contains basic claims for the user's identity. You can extend the set of claims by modifying the SIW [scopes](/docs/reference/api/oidc/#scopes) settings to retrieve custom information about the user. This includes `locale`, `address`, `groups`, and [more](/docs/reference/api/oidc/#scope-values). See [Sign users in to your SPA guide for Vue.js](/docs/guides/sign-into-spa/vue/main/#use-the-access-token) to learn how to use the user's `access_token` to protect routes and validate tokens.