---
title: Configure a Global Session Policy and authentication policies
excerpt: How to configure a Global Session Policy and authentication policies.
layout: Guides
---

<ApiLifecycle access="ie" /><br>

> **Note:** In Okta Classic Engine, Global Session Policy is called the Okta Sign-On Policy and an authentication policy is called an app sign-on policy.

This guide explains what Global Session Policies and authentication policies are used for and how to add and configure them in your [Okta organization](/docs/concepts/okta-organizations/).

---

**Learning outcomes**

* Know the purpose of a Global Session Policy and authentication policies.
* Add and configure a Global Session Policy and authentication policies.

**What you need**

* [Okta Developer Edition organization](https://developer.okta.com/signup)
* [Groups created](/docs/reference/api/groups/) in your org
* An application that you want to assign to an authentication policy
* [Authenticators](https://help.okta.com/okta_help.htm?type=oie&id=csh-configure-authenticators) configured in your org
* A configured [dynamic network zone](https://help.okta.com/okta_help.htm?id=ext_Security_Network)

---

## About authentication policies

Authentication policies are built on IF/THEN rules for app access. IF conditions define the authentication context, like the IP address where a user is signing in, and THEN conditions define the authentication experience, like what assurance factors are required to access an app.

Assurance refers to a level of confidence that the user signing in is also the person who owns the account. This level is measured by the use of one or more authenticators and the [types of factors configured](https://help.okta.com/okta_help.htm?type=oie&id=csh-configure-authenticators). For example, a user who authenticates with a banking app using both a knowledge factor (a password) and a possession factor (an SMS code) has a higher assurance level than a user who authenticates with a shopping app using only one factor. The banking app requires a higher level of assurance than the shopping app, which is why a user signs in with two factors for the banking app and only one factor for the shopping app.

Identity Engine requires that an assurance specified in the Global Session Policy and in the authentication policy be satisfied before a user can access an app. This is a change from the traditional model of authentication, which evaluates one policy depending on whether the user signs in to the org or directly through the app.

A Global Session Policy and an authentication policy control the authentication assurance part of your requirements. [Other policies](/docs/concepts/policies/), such as an MFA enrollment policy, password policy, profile policy, and so on, work together to determine the overall authentication experience.

### Global Session Policy

Global Session Policies help control who can have access and how a user gains access to Okta, including whether they are challenged for additional factors and how long they are allowed to remain signed in before re-authenticating. A Global Session Policy supplies the context necessary for the user to advance to the next authentication step after they are identified by Okta.

You can configure a Global Session Policy to require any of the [factors that you set up](https://help.okta.com/okta_help.htm?type=oie&id=csh-configure-authenticators). Then use the primary and secondary factor conditions in a rule to define which factors are evaluated. For example, add a rule that prompts for additional factors when you want only users who are inside your [corporate network](/docs/reference/api/policy/#network-condition-object) to have access.

> **Note:** If you select **Password/IDP/any factor allowed by app sign on rules** as the primary factor for a rule, you remove the global password requirement from the Global Session Policy and transfer responsibility for defining and enforcing authentication criteria to each of your [authentication policies](#authentication-policy) instead. See [Configure passwordless authentication](https://help.okta.com/okta_help.htm?type=oie&id=ext-passwordless-auth).

You can specify any number of Global Session Policies and the order in which they are executed. If a policy in the list doesn't apply to the user trying to sign in, the system moves to the next policy. There is one required organization-wide policy named Default. By definition, the Default policy applies to all users.

### Authentication policy

In addition to the Global Session Policy, you can configure an authentication policy per app for extra levels of authentication that you want performed before a user gains access to your app. You can also [share authentication policies across multiple apps](link to Graham’s Add apps to a sign-on policy topic).

Each app in your org already has a default authentication policy in place. Use this policy (or create a new one) to determine the additional types of authentication that you want, such as prompting some groups assigned to your app to authenticate with an additional factor.

When you add a new app, it's automatically assigned the shared default policy with a single catch-all rule that allows a user access with only one factor. You can add as many rules to the default policy as you need, but remember that the changes are applied to all new apps.

You don’t have to use the default authentication policy. You can create a new policy specifically for an app, or you can [add an app to another existing shared policy](link to Graham’s Add apps to an app sign-on policy topic). If you decide later to change an app’s sign-on requirements, you can modify its policy or switch to a different shared policy using the [Authentication Policies page](link to Graham’s add a rule to a policy topic).

> **Note:** There can be only one authentication policy per app.

## Configure policies for common scenarios

This section provides step-by-step instructions to configure a Global Session Policy and add a rule to the shared default authentication policy for two of the most common scenarios:

* [Prompt for an additional factor for a group](#prompt-for-an-additional-authenticator-for-a-group)
* [Prompt for an additional factor when a user is outside the US](#prompt-for-an-mfa-factor-when-a-user-is-outside-the-us)

## Prompt for an additional factor for a group

Configure a Global Session Policy to prompt a user for an additional [factor](https://help.okta.com/okta_help.htm?type=oie&id=csh-configure-authenticators) when the user is a member of a certain group.

### Create the policy container

1. In the Admin Console, go to **Security** > **Global Session Policy**.

2. Click **Add New Global Session Policy**.

3. In the **Add Policy** window, enter a **Policy Name**, such as **Require MFA for Contractors**, and then enter a **Policy Description**.

4. In the **Assign to Groups** box, enter the group name that you want to apply the policy to. In this example, we are specifying the **Contractor** group in our org. The group names must already exist before assigning them to a policy.

5. Click **Create Policy and Add Rule**.

### Create the rule

1. In the **Add Rule** window, add a descriptive name for the rule in the **Rule name** box, such as **Require contractors to use MFA once per session**.

2. If there are any users in the **Contractor** group that you want to exclude from the rule, enter them in the **Exclude Users** box.

3. Configure IF conditions, which define the authentication context for the rule. For this use case example, leave the defaults. For other use cases where you want to assign location parameters, you can specify what kind of location prompts authentication in the **IF User’s IP is** drop-down box. For example, prompting a user for a factor when they aren't on the corporate network.

4. Configure THEN conditions, which define the authentication experience for the rule. For this use case example, leave the **Password/IDP** option selected. Additionally, leave **Require secondary factor** selected so that users of the Contractor group are prompted for a secondary factor before they are granted access

    > **Note:** Click the **Multifactor Authentication** link for quick access to the **Authenticators** page to configure the factors that you want to use.

5. Use the option buttons to determine how users are prompted for a secondary factor in a given session. In this example, leave the default of **Per Session** selected.

    You can configure whether the factor prompt is triggered per a device, at every sign-on, or per a session time that you specify:

    * **Per Device:** Provides the option **Do not challenge me on this device again** in the end user MFA challenge dialog. This option allows prompts solely for new devices.
    * **Every Time:** End users are prompted every time they sign in to Okta and can't influence when they are prompted to provide a factor.
    * **Per Session:** Provides the option **Do not challenge me on this device for the next (minutes/hours/days)** in the end user MFA challenge dialog. You specify the **Factor Lifetime** below. When specifying per session, note that sessions have a default lifetime as configured, but sessions always end whenever users sign out of their Okta session.

6. For this use case example, leave the default **Factor Lifetime** of **15 minutes**. Use these fields to specify how much time must elapse before the user is challenged again for the secondary factor.

    The maximum lifetime period is six months. Setting a factor lifetime is a way for end users to sign out for the amount of time noted in the **Factor Lifetime** and not have to authenticate again with MFA at the next sign in. End users must select a box when they sign in to confirm that the setting should be applied. An example is **Do not challenge me on this device for the next 15 minutes**. In this case, after signing out, there is no secondary factor prompt if the user signs in again within 15 minutes of the last sign in with MFA. If users don't select the box, they are always prompted for a secondary factor. The time since the last sign in is noted at the bottom of the End-User Dashboard. However, end users must refresh the page to see the updated value.

7. For this use case example, select **8 hours** for **Session Expires After**. Use these fields to specify the maximum idle time before an authentication prompt is triggered. The maximum allowed time for this option is 90 days. This isn't the total connection time. This is idle time before users see a countdown timer at the 5-minute mark of remaining session time.

    > **Note:** You can also set the [maximum session lifetime value](/docs/reference/api/policy/#signon-session-object) using the Okta APIs. If you previously set this value using the API, you can't exceed that maximum in the Admin Console. Setting a value over the API maximum results in an error.

8. Click **Create Rule**.

> **Note:** After you create a new policy, you must close all active sessions for the new policy to take effect.

## Prompt for an additional factor when a user is outside the US

You may want all default Okta users to provide a password, but you want all Okta users outside of the United States to provide both a password and another factor to access your app. You can use the default authentication policy’s catch-all rule that challenges all users to provide a password. Then, create another rule that challenges all users not within the United States to provide both a password and another factor each time that they sign in.

> **Note:**  You can add as many rules to the default authentication policy that you want, but remember that the changes are applied to all new apps as it is a shared app policy.

### Select the policy and add a rule

Follow these steps to configure another rule for the default authentication policy to prompt a user for an additional factor when the user is outside of the United States.

> **Note:** This example assumes that you have already [set up a Dynamic Zone](https://help.okta.com/okta_help.htm?id=ext_Security_Network). In this example, we are using a dynamic zone defined for IP addresses within the United States.

1. In the Admin Console, select **Security** > **Authentication Policies**.

2. Select **Default Policy** and then **Add rule**. The Add Rule dialog appears.

3. Enter a **Rule name** such as **Prompt for an MFA factor when a user is outside the US**.

4. Configure IF conditions to define the authentication context for the rule. Select **Not in any of the following zones** from the **AND User’s IP is** drop-down list.

    > **Note:** You can click the **Go to Network Zones** link to access the gateway settings that enable your choice of access. A [network zone](https://help.okta.com/okta_help.htm?id=ext_Security_Network) is a security perimeter used to limit or restrict access to a network based on a single IP address, one or more IP address ranges, or a list of geolocations. You can also create network zones using the [Zone API](/docs/reference/api/zones/).

5. In the text box, start typing the dynamic zone name and then select it from the list that appears.

6. Configure THEN conditions to define the authentication experience. For this use case, leave the default of **Allowed after successful authentication** for **THEN Access is**.

7. In the **Re-authentication frequency** section, select **Every sign-in attempt** for both **AND Password re-authentication frequency is** and **AND Re-authentication frequency for all other factors is**.

8. Click **Save**.

> **Note:** You can [use the API](/docs/reference/api/policy/#app-sign-on-policy) to assign an app to an authentication policy. You just need the application ID and the policy ID. Make a `PUT /api/v1/apps/{appId}/policies/{policyId}` request. No HTTP body is necessary for the PUT. Then, to check that the assignment was successful, make a `GET /api/v1/apps/{appId}` request and the response should contain information on the policy associated with the app.

## See also

* [Sign Users Out](/docs/guides/sign-users-out/)
* [Set up self-registration](/docs/guides/set-up-self-service-registration/)
* [Configure an access policy](/docs/guides/configure-access-policy/)
