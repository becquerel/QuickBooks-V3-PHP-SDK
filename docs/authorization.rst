==========
Authorization
==========

This page provides a quick introduction to QuickBooks Online Authorization Protocol: OAuth 1.0a and OAuth 2.0, and how to use it in the SDK. If you have not already installed QuickBooks V3 SDK, head over to the :ref:`installation`
page.


OAuth 1.0a
================

For all developer accounts registered at https://developer.intuit.com before **July.17th, 2017**, they will have OAuth 1.0a protocol default for their apps. 

The developer will see "App Token", "OAuth Consumer Key", and "OAuth Consumer Secret" on the "Keys" tab in the app.

QuickBooks V3 SDK didn't provide a way to generate OAuth 1.0a tokens from OAuth Consumer Key and OAuth Consumer Secret. Developers must generate their own OAuth 1.0a tokens *BEFORE* they use the SDK. 

.. note::

    QuickBooks Online provides an online tool called OAuth 1.0 Playground to help developers generate OAuth 1.0a tokens:
    https://appcenter.intuit.com/Playground/OAuth/IA/ without writing any code.
    For server side web application implementing OAuth 1.0a, please refer here:
    https://intuitdeveloper.github.io/ for sample code. For implementation details, please refer to here:
    https://developer.intuit.com/docs/00_quickbooks_online/2_build/10_authentication_and_authorization/40_oauth_1.0a
    
    
After developer managed to get OAuth 1.0a tokens, provides it to the DataService object:

.. code-block:: php

    use QuickBooksOnline\API\DataService\DataService;

    // Prep Data Services
    $dataService = DataService::Configure(array(
         'auth_mode' => 'oauth1',
         'consumerKey' => "The OAuth Consumer key Value from Keys tab",
         'consumerSecret' => "The OAuth Consumer secret Value from Keys tab",
         'accessTokenKey' => "The OAuth 1.0a access token returned from QuickBooks Online",
         'accessTokenSecret' => "The OAuth 1.0a access token secret retruned from QuickBooks Online",
         'QBORealmID' => "The Company ID which the app wants to access",
         //If you are using Development Keys, use Development here. If you are using Production Keys, use Production.
         'baseUrl' => "Development/Production"
    ));
    
Here is an actual example for configuring OAuth 1.0a value for a sandbox Company:

.. code-block:: php

    use QuickBooksOnline\API\DataService\DataService;

    // Prep Data Services
    $dataService = DataService::Configure(array(
         'auth_mode' => 'oauth1',
         'consumerKey' => "qyprdUSoVpIHrtBp0eDMTHGz8UXuSz",
         'consumerSecret' => "TKKBfdlU1I1GEqB9P3AZlybdC8YxW5qFSbuShkG7",
         'accessTokenKey' => "qyprdxccscoNl7KRbUJoaJQIhUvyXRzD9tNOlXn4DhRDoj4g",
         'accessTokenSecret' => "JqkHSBKzNHbqjMq0Njbcq8fjgJSpfjMvqHVWnDOW",
         'QBORealmID' => "193514464689044",
         'baseUrl' => "Development"
    ));    

OAuth 2.0
================

Most recent QucikBooks Online apps will have OAuth 2.0 as their default protocol. If you see "Client ID" and "Client Secret" under your "Keys" tab, then your app is using OAuth 2.0 protocol. QuickBooks V3 SDK provides methods to generate OAuth 2.0a tokens, and methods to use them like OAuth 1.0a provided.


Generate OAuth 2.0 Tokens
---------------------

In order for the SDK to generate OAuth 2.0 tokens for the app, developers will need to provide necessary parameters. All these parameters can be found in the app or our documentation here: https://developer.intuit.com/docs/00_quickbooks_online/2_build/10_authentication_and_authorization/10_oauth_2.0 

.. code-block:: php

    use QuickBooksOnline\API\DataService\DataService;

    // Prep Data Services
    $dataService = DataService::Configure(array(
          'auth_mode' => 'oauth2',
          'ClientID' => "Client ID from the app's keys tab",
          'ClientSecret' => "Client Secret from the app's keys tab",
          'RedirectURI' => "The redirect URI provided on the Redirect URIs part under keys tab",
          'scope' => "com.intuit.quickbooks.accounting or com.intuit.quickbooks.payment",
          'baseUrl' => "Development/Production"
    ));
    
After we have provided necessary parameters, get the OAuth2LoginHelper from the DataService Object:

.. code-block:: php

   $OAuth2LoginHelper = $dataService->getOAuth2LoginHelper();

The OAuth2LoginHelper will help developers to complete all the neccesary steps for retreiving OAuth 2 tokens. 

First, use the OAuth2LoginHelper to generate Authorization URL:

.. code-block:: php

   $authorizationUrl = $OAuth2LoginHelper->getAuthorizationCodeURL();

display this URL to your clients and they will click the "Authorize" button to authorize your app. 

.. note::

    Use 
    .. code-block:: php
    
        header("Location: ".$authorizationUrl);

    to display the authorization screen to your customers. Do not use cURL.

Once the clients have authorized the app, an authorization code and realmID will be returned to the ReditrectURI. Provide these parameters to exchange an OAuth 2 Access Token:

.. code-block:: php

   $accessToken = $OAuth2LoginHelper->exchangeAuthorizationCodeForToken("authorizationCode", "RealmID");

update the DataService object and the app is ready to make API calls with OAuth 2 Tokens. 

.. code-block:: php

   $dataService->updateOAuth2Token($accessToken);

Use OAuth 2.0 Tokens
---------------------

If developers have already retrieved OAuth 2 tokens, they can simply provide it to DataService. It is very similar to OAuth 1.0a, just change the auth_mode from oauth1 to oauth 2.

.. code-block:: php

    use QuickBooksOnline\API\DataService\DataService;

    // Prep Data Services
    $dataService = DataService::Configure(array(
         'auth_mode' => 'oauth2',
         'ClientID' => "Client ID from the app's keys tab",
         'ClientSecret' => "Client Secret from the app's keys tab",
         'accessTokenKey' => 'OAuth 2 Access Token',
         'refreshTokenKey' => "OAuth 2 Refresh Token",
         'QBORealmID' => "The Company ID which the app wants to access",
         'baseUrl' => "Development/Production"
    ));
    
.. note::

    Similar to OAuth 1.0 Playground, QuickBooks Online also provides OAuth 2.0 Playground to help developers generate OAuth 
    2.0 tokens without writing code. To access OAuth 2.0 Playground, you will need to log into https://developer.intuit.com,
    go to your app' dashboard and click "Test connect to app (OAuth)" link there.
   


