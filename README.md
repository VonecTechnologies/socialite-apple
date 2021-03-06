<h1 align="center">Socialite - Apple</h1>
<p align="center">
<a href="https://packagist.org/packages/ahilmurugesan/socialite-apple"><img src="https://poser.pugx.org/ahilmurugesan/socialite-apple/d/total.svg" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/ahilmurugesan/socialite-apple"><img src="https://poser.pugx.org/ahilmurugesan/socialite-apple/v/stable.svg" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/ahilmurugesan/socialite-apple"><img src="https://poser.pugx.org/laravel/framework/license.svg" alt="License"></a>
</p>

---

## 1. Installation

```bash
// This assumes that you have composer installed globally
composer require ahilmurugesan/socialite-apple
```

## 2. Service Provider

* Remove `Laravel\Socialite\SocialiteServiceProvider` from your `providers[]` array in `config\app.php` if you have added it already.

* Add `\SocialiteProviders\Manager\ServiceProvider::class` to your `providers[]` array in `config\app.php`.

For example:

``` php
'providers' => [
    // a whole bunch of providers
    // remove 'Laravel\Socialite\SocialiteServiceProvider',
    \SocialiteProviders\Manager\ServiceProvider::class, // add
];
```

* Note: If you would like to use the Socialite Facade, you need to [install it.](https://github.com/laravel/socialite)

## 3. Event Listener

* Add `SocialiteProviders\Manager\SocialiteWasCalled` event to your `listen[]` array  in `app/Providers/EventServiceProvider`.

* Add your listeners (i.e. the ones from the providers) to the `SocialiteProviders\Manager\SocialiteWasCalled[]` that you just created.

* The listener that you add for this provider is `'Ahilan\\Apple\\AppleExtendSocialite@handle',`.

* Note: You do not need to add anything for the built-in socialite providers unless you override them with your own providers.

For example:

```php
/**
 * The event handler mappings for the application.
 *
 * @var array
 */
protected $listen = [
    \SocialiteProviders\Manager\SocialiteWasCalled::class => [
        // add your listeners (aka providers) here
        'Ahilan\\Apple\\AppleExtendSocialite@handle',
    ],
];
```

#### Reference

* [Laravel docs about events](http://laravel.com/docs/events)

## 4. Configuration Setup

You will need to add an entry to the services configuration file so that after config files are cached for usage in production environment (Laravel command `artisan config:cache`) all config is still available.

#### Add to `config/services.php`.

```php
"apple" => [    
  "client_id" => env("APPLE_CLIENT_ID"),  
  "client_secret" => env("APPLE_CLIENT_SECRET"),  
  "redirect" => env("APPLE_REDIRECT_URI"),
  "key_id" => env("APPLE_KEY_ID"),  
  "team_id" => env("APPLE_TEAM_ID"),  
  "auth_key" => env("APPLE_AUTH_KEY"),  
  "client_secret_updated_at" => env("APPLE_CLIENT_SECRET_UPDATED_AT"),  
  "refresh_token_interval_days" => env("APPLE_REFRESH_TOKEN_INTERVAL_DAYS"),  
],
```
To set up the required environment variables you can use the following artisan command which comes with this package. 

```
php artisan socialite:apple
```

Please watch the following video to understand the flow to obtain the required Sign in with Apple credentials.

<a href="https://youtu.be/rgHL2JBxjJ0">![siwa-video-cover](https://user-images.githubusercontent.com/648370/80411038-2653b900-88e9-11ea-953b-d34a2272000a.png)</a>
The command will prompt you the required values which can be acquired by following the [setup video](https://youtu.be/rgHL2JBxjJ0).  You need to provide the following keys.
 1. Team ID
 2. Key ID
 3. Client ID
 4. Auth Key ( file name of p8 auth file, located inside storage/app/ ) Example: AuthKey_SAMPKEY.p8
 5. Redirect URI ( fully qualified secure callback url ) Example: https://website.com/socialite/apple/callback
 6. Token Refresh Interval ( in days )

Client Secret will be automatically generated and added to the .env file by using the above command. 

> The expiration time registered claim key, the value of which must not be greater than 15777000 (6 months in seconds) from the Current Unix Time on the server.

[Sign in with Apple](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens) Client Secret expiration time cannot be greater than six months. Hence it is advisible to refresh the Client Secret atleast once in six months after creation. You can adjust the Token Refresh Interval. There is a scheduled task which comes along with this package which will ensure that the Client Token is refreshed automatically. Please do ensure that you have enabled [Task Scheduling](https://laravel.com/docs/master/scheduling#introduction)

To manually refresh the Client Secret, please run the following command
```
php artisan socialite:apple --refresh
```



## 5. Usage

* [Laravel docs on configuration](http://laravel.com/docs/master/configuration)

* You should now be able to use it like you would regularly use Socialite (assuming you have the facade installed):

```php
// authorize with provider
return Socialite::with('apple')->redirect();

// fetch user after callback
$user = Socialite::with('apple')->user();

// fetch user using token ( token from apple authentication )
$token = "eyJraWQiOiJlWGF1bm1MIiwiYWxnIjoiUlMyNTYifQ.eyJpc3MiOiJodHRwczovL2FwcGxlaWQuYXBwbGUuY29tIiwiYXVkIjoiY29tLnZvbmVjLnNpd2EuYXBpIiwiZXhwIjoxNTg3OTI2MjAzLCJpYXQiOjE1ODc5MjU2MDMsInN1YiI6IjAwMTcxMC44NThkN2NhNWUwZDg0MWI5ODFiNGVkYWY2NWM0M2ZmNi4xOTMyIiwiYXRfaGFzaCI6IjRHZFprR0k2X2Q3Qk5xMFFJTkhKZEEiLCJlbWFpbCI6ImFoaWxtdXJ1Z2VzYW5AZ21haWwuY29tIiwiZW1haWxfdmVyaWZpZWQiOiJ0cnVlIiwiYXV0aF90aW1lIjoxNTg3OTI1NjAxLCJub25jZV9zdXBwb3J0ZWQiOnRydWV9.ciXdwwkySnG-Ne_l9NqxuLkDPyptUVvJ_Puk10LSsXNEtLBAijskQhIjwi3HYsEXNLdlbMGfJ25rnlMWu93RoqYJFo_u_rFjH_4Xt9E_ddnqY147yZvVw5k912FtXabQSl2bFiR7yrzuQvznxyAiYFP9v9HvXyTcYS2ki6ISdPjmTyb927yWyGDx-aigksV752toAA8XXmjjEyi01eY-wng4CaV4mxjJU_bQSpnh6zGLpmI-lxqBIfSbvW1ukMDh9VW7fIRq9l3yFba91TAT9oBv7QQVcEAU7jHNzKX3qU7JvCfr7d2UUXFVkOxYZFz1HuPHB5C9QuYn5TtFUb2ozw";
$user = Socialite::with('apple')->userFromToken($token));
```

* [Laravel docs on Socialite](https://laravel.com/docs/master/socialite)
* [Demo Repo](https://github.com/VonecTechnologies/socialite-apple-sample/)

### Lumen Support

You can use Socialite providers with Lumen.  Just make sure that you have facade support turned on and that you follow the setup directions properly.

**Note:** If you are using this with Lumen, all providers will automatically be stateless since **Lumen** does not keep track of state.

Also, configs cannot be parsed from the `services[]` in Lumen.  You can only set the values in the `.env` file as shown exactly in this document.  If needed, you can
  also override a config (shown below).

### Stateless

* You can set whether or not you want to use the provider as stateless.  Remember that the OAuth provider (Twitter, Tumblr, etc) must support whatever option you choose.

**Note:** If you are using this with Lumen, all providers will automatically be stateless since **Lumen** does not keep track of state.

```php
// to turn off stateless
return Socialite::with('apple')->redirect();

// to use stateless
return Socialite::with('apple')->stateless()->redirect();
```

### Overriding a config

If you need to override the provider's environment or config variables dynamically anywhere in your application, you may use the following:

```php
$clientId = "secret";
$clientSecret = "secret";
$redirectUrl = "http://yourdomain.com/api/redirect";
$additionalProviderConfig = ['site' => 'meta.stackoverflow.com'];
$config = new \SocialiteProviders\Manager\Config($clientId, $clientSecret, $redirectUrl);
return Socialite::with('apple')->setConfig($config)->redirect();
```

### Retrieving the Access Token Response Body

Laravel Socialite by default only allows access to the `access_token`.  Which can be accessed
via the `\Laravel\Socialite\User->token` public property.  Sometimes you need access to the whole response body which
may contain items such as a `refresh_token`.

You can get the access token response body, after you called the `user()` method in Socialite, by accessing the property `$user->accessTokenResponseBody`;

```php
$user = Socialite::driver('apple')->user();
$accessTokenResponseBody = $user->accessTokenResponseBody;
```

#### Reference

* [Laravel Socialite Docs](https://laravel.com/docs/socialite)
* [Sign in with Apple Docs](https://developer.apple.com/documentation/sign_in_with_apple)
