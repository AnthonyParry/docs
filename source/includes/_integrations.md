# Integrations

Use Scaphold's integration infrastructure to tie third-party services into your API. When you add an integration, we will automatically update your API with new functionality. Integrations range in purpose. Want notifications? Add iOS Push. Want payments? Integrate Stripe. Need social auth? Add it with Auth0. Our collection of integrations is always expanding so please let us know if you have any specific requests.

Integrations come in many shapes and sizes. Some integrations like Slack and Webhooks are powered by events in your API while others like Mailgun will expose their functionality by adding queries and mutations to your GraphQL API.

We've got some exciting new integrations coming out soon! Please [let us know](mailto:support@scaphold.io) if you have any specific requests.

## Search

Use Algolia to extend your app's search capabilities, and query your data seamlessly as your data changes.

1. Create a free [Algolia](https://www.algolia.com/) account. This will help you manage your search indices and monitor your usage. Once you've created your account, you'll receive an Application ID and an API Key.

        *Note: For different apps on Scaphold, you'll need to create a new Algolia account with different keys since new search indices will interfere with existing ones.*

2. Configure **Algolia** in Scaphold from the [Integrations Portal](https://scaphold.io/#/apps/integrations) to enable the checkboxes in the Schema Designer that will allow you to index data of a particular type.

        <img class="news-how-to-img" src="images/news/search_enable.png" alt="Configure your Algolia integration">

3. As soon as you enable search on a particular type, Scaphold will index each existing piece of data for that type into Algolia's system and future CREATE, UPDATE, and DELETE commands will be reflected in both Scaphold and Algolia for search. These commands will be the same as before; however you now have a **new query for each searchable type under Viewer** that will allow you to search for query terms along with optional parameters.

        <img class="news-how-to-img" src="images/news/search_query.png" alt="Search for a term">

## Social Auth

Use **Auth0** to extend your app's login and registration flow painlessly through your favorite social authentication services.

1. Create a free [Auth0](https://auth0.com/) account. This will help you manage your app credentials like client IDs and secrets for your OAuth providers. By connecting your apps on your social accounts like Facebook, Google, and Twitter, you'll then have the correct account credentials to utilize these services for your authentication flow.

        <img src="images/news/configure_auth0_account-min.png" alt="Configure your Auth0 apps.">

2. Configure Auth0 in Scaphold from the [Integrations Portal](https://scaphold.io/#/apps/integrations) to include the OAuth providers that you plan to use for your app. This will enable a new mutation called **loginUserWithAuth0Social** which you can use to log in users with connected OAuth providers.

3. In your client app, you'll likely be using a client SDK to handle user login. For instance, you can use the [Facebook SDK for React Native](https://github.com/facebook/react-native-fbsdk) to ask your users to log in with their existing Facebook accounts. Once this succeeds, you'll receive an **access token** to send to Scaphold.

        <img src="images/news/configure_auth0-min.png" alt="Configure your Auth0 integration to include auth providers">

        <img src="images/news/get_access_token-min.png" alt="Get access token">

        **UPDATE:** Mutation name is now called loginUserWithAuthOSocial

4. After Scaphold verifies the access token with the OAuth provider (i.e. Facebook), we'll pass back the **JSON Web Token (JWT)** that you'll need to add to your authorization header for future requests. That way, you will be authorized to make future requests through Scaphold and it will also provide us the capability to work on that user's behalf to access the OAuth provider's resources. For instance, we could authorize you to access your Facebook friends and their public profile information.

        <img src="images/news/insert_jwt_header-min.png" alt="Set JWT token in header">

5. In addition, if you've logged in already and you make a request to **loginUserWithAuth0Social** again but with a new OAuth provider (i.e. Google), Scaphold will link your two accounts together, since we know the requests being made belong to one user. Now, you'll have access to both Facebook and Google information using Facebook's and Google's account credentials.

        <img src="images/news/link_google_user-min.png" alt="Link Google account">

        **UPDATE:** Mutation name is now called loginUserWithAuth0Social

6. If you'd like to use Auth0 Lock as a client-side SDK to manage your authentication, you can do so with the **loginUserWithAuth0Lock** mutation. This accepts the identity parameter that the client receives from a successful Auth0 Lock login in the profile variable. These work in sync with all other Scaphold basic and social authentication flows so you can manage your users how you want.

## Payments

Hook into **Stripe** to instantly add advanced payment methods to your application.

The Stripe integration allows you to quickly and easily link your company's Stripe account with your Scaphold API.

1. Head to [Stripe](https://stripe.com/) and create a free account to get started.
2. Upload your secret key and publishable key via our [Integrations Portal](https://scaphold.io/#/apps/integrations).
3. Rejoice as we have just added tons of new payments functionality to your API.

## Push Notifications

Add iOS push notifications in an instant.

1. Follow [this tutorial](https://www.raywenderlich.com/123862/push-notifications-tutorial) to create an SSL certificate and PEM file
2. Add the iOS integration and upload your SSL cert and PEM file through our [Integrations Portal](https://scaphold.io/#/apps/integrations).
3. Immediately get access to iOS push notifications via your apps GraphQL API.

## Email

Use **Mailgun** to send and manage email from within your application. Bind emails to events to automate your workflow.

1. Create a free [Mailgun](https://mailgun.com/) account.
2. Add the Mailgun integration and enter you API key and domain name in our integrations portal.
3. Start sending emails and managing mailing lists from your Scaphold API!
