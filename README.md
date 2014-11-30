# Oops/TotpAuthenticator

Oops/TotpAuthenticator implements the [TOTP algorithm](http://tools.ietf.org/html/rfc6238) that lets you easily set up a two-factor authentication mechanism. The TOTP, in short, generates a pseudo-random six-digit number based on current Unix time and given seed (a base32 string of at least 16 characters). The seed is shared between you and mobile app and passed to the app via a QR code. Whenever you need to authenticate the user, ask him to enter the code generated by the application, and verify it against the code generated by your server.


## Installation and requirements

```bash
$ composer require oops/totp-authenticator:~2.0
```

Oops/TotpAuthenticator requires PHP >= 5.4 (because I'm too lazy to write `array()`).


## Usage

If you use Nette's DI container, you can easily integrate Oops/TotpAuthenticator with a few lines of configuration:

```
extensions:
	totpAuth: Oops\TotpAuthenticator\DI\TotpAuthenticatorExtension

totpAuth:
	timeWindow: 1
	issuer: MyApp
```

Otherwise, you can directly instantiate `Oops\TotpAuthenticator\Security\TotpAuthenticator` and optionally configure it:
 
```php
$totpAuthenticator = (new Oops\TotpAuthenticator\Security\TotpAuthenticator)
   ->setIssuer('MyApp')
   ->setTimeWindow(1);
```

The `timeWindow` option sets a benevolence that compensates for possible differences between your server's time and the app's time. The default value is 1, which means the code for previous or next 30-second block would be also considered valid. You can set it to zero if you want to be super strict, but I'd strongly recommend not setting it to a higher value than one.

The `issuer` is optional, but is quite useful if you use some generic value as the user's account name, such as their email address. As multiple services can use that same value as a way of identifying the user, you should provide `issuer` to distinguish your app's code in the TOTP app.


### Setting up 2FA

First, you need to generate a secret - a base32 string - that is shared between you and the application. `TotpAuthenticator` provides a method for that. The secret must be unique to the user and should thus be stored somewhere with other user data, e.g. in the database. Just remember to encrypt it as it is a very sensitive piece of information.

```php
$secret = $totpAuthenticator->getRandomSecret();
```

Then, you can display the secret and unique account name (e.g. user's email address) to the user, or, more conveniently, provide them with a QR code containing the URI in a specific format. Again, `TotpAuthenticator` can build the URI for you:

```php
$uri = $totpAuthenticator->getTotpUri($secret, $accountName);
```

(To generate the QR code, you can use `qrencode` and the [wrapper](https://github.com/Kdyby/QrEncode) for Nette.)


### Verification

Once the user has set up the account in their TOTP app, you can ask them to enter the 6-digit code and easily verify it against the secret:

```php
if ($totpAuthenticator->verifyCode($code, $secret)) {
	// successfully verified

} else {
	// incorrect code
}
```
