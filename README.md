# Google Auth Library for PHP

<dl>
  <dt>Homepage</dt><dd><a href="http://www.github.com/google/google-auth-library-php">http://www.github.com/google/google-auth-library-php</a></dd>
  <dt>Reference Docs</dt><dd><a href="https://googleapis.github.io/google-auth-library-php/master/">https://googleapis.github.io/google-auth-library-php/master/</a></dd>
  <dt>Authors</dt>
    <dd><a href="mailto:betterbrent@google.com">Brent Shaffer</a></dd>
    <dd><a href="mailto:dsupplee@google.com">David Supplee</a></dd>
  <dt>Copyright</dt><dd>Copyright © 2020 Google LLC</dd>
  <dt>License</dt><dd>Apache 2.0</dd>
</dl>

## Description

This is Google's officially supported PHP client library for using OAuth 2.0
authorization and authentication with Google APIs.

### Installing via Composer

The recommended way to install the google auth library is through
[Composer](http://getcomposer.org). Run the Composer command to install the latest stable version:

```bash
composer require google/auth
```

## Application Default Credentials

This library provides an implementation of
[application default credentials][application default credentials] for PHP.

The Application Default Credentials provide a simple way to get authorization
credentials for use in calling Google APIs.

They are best suited for cases when the call needs to have the same identity
and authorization level for the application independent of the user. This is
the recommended approach to authorize calls to Cloud APIs, particularly when
you're building an application that uses Google Compute Engine.

#### Download your Service Account Credentials JSON file

To use `Application Default Credentials`, You first need to download a set of
JSON credentials for your project. Go to **APIs & Services** > **Credentials** in
the [Google Developers Console][developer console] and select
**Service account** from the **Add credentials** dropdown.

> This file is your *only copy* of these credentials. It should never be
> committed with your source code, and should be stored securely.

Once downloaded, store the path to this file in the
`GOOGLE_APPLICATION_CREDENTIALS` environment variable.

```php
putenv('GOOGLE_APPLICATION_CREDENTIALS=/path/to/my/credentials.json');
```

> PHP's `putenv` function is just one way to set an environment variable.
> Consider using `.htaccess` or apache configuration files as well.

#### Enable the API you want to use

Before making your API call, you must be sure the API you're calling has been
enabled. Go to **APIs & Auth** > **APIs** in the
[Google Developers Console][developer console] and enable the APIs you'd like to
call. For the example below, you must enable the `Drive API`.

#### Call the APIs

As long as you update the environment variable below to point to *your* JSON
credentials file, the following code should output a list of your Drive files.

```php
use Google\Auth\GoogleAuth;
use Google\Auth\Http\CredentialsClient;
use GuzzleHttp\Psr7\Request;

// specify the path to your application credentials
putenv('GOOGLE_APPLICATION_CREDENTIALS=/path/to/my/credentials.json');

// define the scopes for your API call
$scopes = ['https://www.googleapis.com/auth/drive.readonly'];

// create the auth client
$googleAuth = new GoogleAuth();

// authorize an http client
$credentials = $googleAuth->makeCredentials(['scope' => $scopes]);
$client = new CredentialsClient($credentials);

// make the request
$request = new Request('GET', 'https://www.googleapis.com/drive/v2/files');
$response = $client->send($request);

// show the result!
print_r((string) $response->getBody());
```

#### Call using an ID Token
If your application is running behind Cloud Run, or using Cloud Identity-Aware
Proxy (IAP), you will need to fetch an ID token to access your application. For
this, use the static method `getIdTokenMiddleware` on
`GoogleAuth`.

```php
use Google\Auth\GoogleAuth;
use GuzzleHttp\Psr7\Request;

// specify the path to your application credentials
putenv('GOOGLE_APPLICATION_CREDENTIALS=/path/to/my/credentials.json');

// Provide the ID token audience. This can be a Client ID associated with an IAP application,
// Or the URL associated with a CloudRun App
//    $targetAudience = 'IAP_CLIENT_ID.apps.googleusercontent.com';
//    $targetAudience = 'https://service-1234-uc.a.run.app';
$targetAudience = 'YOUR_ID_TOKEN_AUDIENCE';

// create the auth client
$googleAuth = new GoogleAuth();

// authorize an http client
$credentials = $googleAuth->makeCredentials([
    'targetAudience' => $targetAudience
]);
$client = new CredentialsClient($credentials);

// make the request
$request = new Request('GET', 'https://YOUR_PROTECTED_RESOURCE');
$response = $client->send($request);

// show the result!
print_r((string) $response->getBody());
```

For invoking Cloud Run services, your service account will need the
[`Cloud Run Invoker`](https://cloud.google.com/run/docs/authenticating/service-to-service)
IAM permission.

For invoking Cloud Identity-Aware Proxy, you will need to pass the Client ID
used when you set up your protected resource as the target audience. See how to
[secure your IAP app with signed headers](https://cloud.google.com/iap/docs/signed-headers-howto).

#### Verifying JWTs

If you are [using Google ID tokens to authenticate users][google-id-tokens], use
the `Google\Auth\AccessToken` class to verify the ID token:

```php
use Google\Auth\GoogleAuth;

$googleAuth = new GoogleAuth();
$googleAuth->verify($idToken);
```

If your app is running behind [Google Identity-Aware Proxy][iap-id-tokens]
(IAP), you can verify the ID token coming from the IAP server by pointing to the
appropriate certificate URL for IAP. This is because IAP signs the ID
tokens with a different key than the Google Identity service:

```php
use Google\Auth\GoogleAuth;

$auth = new GoogleAuth();
$auth->verify($idToken, [
  'certsLocation' => GoogleAuth::IAP_JWK_URL
]);
```

[google-id-tokens]: https://developers.google.com/identity/sign-in/web/backend-auth
[iap-id-tokens]: https://cloud.google.com/iap/docs/signed-headers-howto

## License

This library is licensed under Apache 2.0. Full license text is
available in [COPYING][copying].

## Contributing

See [CONTRIBUTING][contributing].

## Support

Please
[report bugs at the project on Github](https://github.com/google/google-auth-library-php/issues). Don't
hesitate to
[ask questions](http://stackoverflow.com/questions/tagged/google-auth-library-php)
about the client or APIs on [StackOverflow](http://stackoverflow.com).

[google-apis-php-client]: https://github.com/google/google-api-php-client
[application default credentials]: https://developers.google.com/accounts/docs/application-default-credentials
[contributing]: https://github.com/google/google-auth-library-php/tree/master/.github/CONTRIBUTING.md
[copying]: https://github.com/google/google-auth-library-php/tree/master/COPYING
[Guzzle]: https://github.com/guzzle/guzzle
[Guzzle 5]: http://docs.guzzlephp.org/en/5.3
[developer console]: https://console.developers.google.com
