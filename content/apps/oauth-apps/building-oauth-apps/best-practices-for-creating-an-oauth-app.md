---
title: Best practices for creating an OAuth app
shortTitle: Best practices
intro: 'Follow these best practices to improve the security and performance of your {% data variables.product.prodname_oauth_app %}.'
versions:
  fpt: '*'
  ghes: '*'
  ghec: '*'
topics:
  - OAuth apps
---

## Use a {% data variables.product.prodname_github_app %} instead

If possible, consider using a {% data variables.product.prodname_github_app %} instead of an {% data variables.product.prodname_oauth_app %}. In general, {% data variables.product.prodname_github_apps %} are preferred over {% data variables.product.prodname_oauth_apps %}. {% data variables.product.prodname_github_apps %} use fine-grained permissions, give the user more control over which repositories the app can access, and use short-lived tokens. These properties can harden the security of your app by limiting the damage that could be done if your app's credentials are leaked.

Similar to {% data variables.product.prodname_oauth_apps %}, {% data variables.product.prodname_github_apps %} can still use OAuth 2.0 and generate a type of OAuth token (called a user access token) and take actions on behalf of a user. However, {% data variables.product.prodname_github_apps %} can also act independently of a user.

For more information about {% data variables.product.prodname_github_apps %}, see [AUTOTITLE](/apps/creating-github-apps/setting-up-a-github-app/about-creating-github-apps).

For more information about migrating an existing {% data variables.product.prodname_oauth_app %} to a {% data variables.product.prodname_github_app %}, see [AUTOTITLE](/apps/creating-github-apps/guides/migrating-oauth-apps-to-github-apps).

## Use minimal scopes

Your {% data variables.product.prodname_oauth_app %} should only request the scopes that the app needs to perform its intended functionality. If any tokens for your app become compromised, this will limit the amount of damage that can occur. For more information, see [AUTOTITLE](/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps).

## Authorize thoroughly and durably

After signing in a user, app developers must take additional steps to ensure that the user is meant to have access to the data in your system. Each sign in requires fresh checks around their memberships, access, and their current SSO status.

### Use the durable, unique `id` to store the user

{% data reusables.apps.best-practice-use-durable-id %}

### Validate organization access for every new authentication

{% data reusables.apps.best-practice-validate-org-access %}

### Store user data with organizational and enterprise contexts

{% data reusables.apps.best-practice-store-data-with-context %}

### Verify a user's access to your app

{% ifversion ghec %}An {% data variables.product.prodname_oauth_app %} created by a {% data variables.enterprise.prodname_managed_user %} or {% data variables.enterprise.prodname_emu_org %} can only be accessed by members of the enterprise that owns those accounts. Otherwise, your{% else %}Your{% endif %} OAuth app can be accessed by users outside your organization or enterprise. If you intend an app to be used only by members of your organization or enterprise, you should check the user's membership status when the user signs in to your app.

To find the list of organizations a user is a member of, you can use the "List organizations for the authenticated user" endpoint. Then you can validate this list against a list of approved organizations for your app. For more information, see [AUTOTITLE](/rest/orgs/orgs#list-organizations-for-the-authenticated-user).

## Secure your app's credentials

With a client secret, your app can authorize a user and generate user access tokens. These tokens can be used to make API requests on behalf of a user.

You must store your app's client secret and any generated tokens securely. The storage mechanism and its relative security depends on your integrations architecture and the platform that it runs on. In general, you should use a storage mechanism that is intended to store sensitive data on the platform that you are using.

### Client secrets

Client secrets are required to generate user access tokens for your app, unless your app uses the device flow. For more information, see [AUTOTITLE](/apps/oauth-apps/building-oauth-apps/authorizing-oauth-apps#device-flow).

If your app is a confidential client, meaning it can safely keep the client secret secure, consider storing your client secret in a key vault, such as [Azure Key Vault](https://azure.microsoft.com/products/key-vault), or as an encrypted environment variable or secret on your server.

If your app is a public client (a native app that runs on the user's device, CLI utility, or single-page web application), you cannot secure your client secret. You will have to ship the client secret in the application's code, and you should use PKCE to better secure the authentication flow. You should use caution if you plan to gate access to your own services based on tokens generated by your app because public clients are trivially spoofable - anyone can reuse your app's client ID to sign in.

#### Don't enable device flow without reason

It is preferable to use the authorization code with PKCE over the device flow, if you are concerned about using the client secret in a public client. The device flow does not require redirect URIs at all, which means that an attacker can use the device flow to remotely impersonate your app as part of a phishing attack. For this reason, do not enable the device flow for your application unless you are using the app in a constrained environment (CLIs, IoT devices, or headless systems).

### User access tokens

If your app is a website or web app, you should encrypt the tokens on your back end and ensure there is security around the systems that can access the tokens. Consider storing refresh tokens in a separate place from active access tokens.

If your app is a native client, client-side app, or runs on a user device (as opposed to running on your servers), you may not be able to secure tokens as well as an app that runs on your servers. You should store tokens via the mechanism recommended for your app's platform, and keep in mind that the storage mechanism may not be fully secure.

## Use the appropriate token type

{% data variables.product.prodname_oauth_apps %} can generate user access tokens in order to make authenticated API requests. Your app should never use a {% data variables.product.pat_generic %} or {% data variables.product.company_short %} password to authenticate.

## Make a plan for handling security breaches

You should have a plan in place so that you can handle any security breaches in a timely manner.

In the event that your app's client secret is compromised, you will need to generate a new secret, update your app to use the new secret, and delete your old secret.

In the event that user access tokens are compromised, you should immediately revoke these tokens. For more information, see [AUTOTITLE](/rest/apps/oauth-applications#delete-an-app-token).

## Conduct regular vulnerability scans

{% data reusables.apps.app-scans %}

## Choose an appropriate environment

If your app runs on a server, verify that your server environment is secure and that it can handle the volume of traffic that you expect for your app.

## Use services in a secure manner

{% data reusables.apps.app-services %}

## Add logging and monitoring

{% data reusables.apps.apps-logging %}

## Enable data deletion

If your app is available to other users, you should give users a way to delete their data. Users should not need to email or call a support person in order to delete their data.

{% ifversion fpt or ghec %}

## Further reading

* [AUTOTITLE](/apps/publishing-apps-to-github-marketplace/creating-apps-for-github-marketplace/security-best-practices-for-apps)
* [AUTOTITLE](/apps/publishing-apps-to-github-marketplace/creating-apps-for-github-marketplace/customer-experience-best-practices-for-apps)

{% endif %}
