Has
===

**Document still in progress.**

Has (pronounced "Hash" because it sounds cute) is a flexible validation package for laravel. Heavily inspired by the likes of [Respect\Validation](https://github.com/Respect/Validation), it's meant to be highly configurable and malleable to specific needs.

There are plenty of top-notch validators out there, but they each have their own idea of how to report, handle and format responses. Has originated from peckish user experience designers who want very specific ways of dealing with validation issues, and fussy analysts who demand detailed data regarding what and where stuff is failing.

Has' language style is very opinionated, and we like it that way. We want to keep it succinct, but also be clear and readable.

Getting Started
============

```php
$emailValidator = Has::email()->under(100)->over(2);
$emailValidator->check('example.com');
```

Validators are assumed required unless explicitly stated:

```php
$addressValidator = Has::string()->under(255)->optional();
```

Catching errors:

```php
$ageValidator = Has::int()->over(18);

try {
  $ageValidator->check(15);
} catch (ValidationException $e) {
  echo($e->getMessage());
}

// would print:

"Integer value 15 is under the minimum of 18"
```

We can, of course, customize this. Ideally you'd want to put this string into a string table or configuration file to keep it away from the actual valdiation class.

```php
$ageValidator = Has::int()->over(18);

try {
  $ageValidator->check(15);
} catch (ValidationException $e) {
  echo($e->getMessage([
    'range' => 'You need to be over 18 to ride this roller coaster.'
  ]));
}

```

Sometimes we'll want to do different things when certain errors are found in the passed data, so we can catch for specific exceptions. Now, this is tricky, because this error can be *within* the bigger exception being thrown, so we need to do this instead:

```php
$usernameValidator = Has::alphanumeric()->over(5)->under(20)->allow([' ']); // alphanumeric plus spaces

try {
  $usernameValidator->check("J-D"); // fails because it's too short and has a dash
} catch (ValidationException $e) {
  try {
    $e->rethrow('whitelist'); // alphanumeric and allow rules are called "whitelist" rules
  } catch (WhitelistException $e) {
    // do stuff
  }
  
  try {
    $e->rethrow('range');
  } catch (RangeException $e) {
    // do stuff
  }
  
  echo $e->getMessages();
}
```











