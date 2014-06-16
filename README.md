Has
===

**Document still in progress.**

Has (pronounced "Hash" because it sounds cute) is a flexible validation package for laravel. Heavily inspired by the likes of [Respect\Validation](https://github.com/Respect/Validation), it's meant to be highly configurable and malleable to specific needs.

There are plenty of top-notch validators out there, but they each have their own idea of how to report, handle and format responses. Has originated from peckish user experience designers who want very specific ways of dealing with validation issues, and fussy analysts who demand detailed data regarding what and where stuff is failing.

Has' language style is very opinionated, and we like it that way. We want to keep it succinct, but also be clear and readable. A lot of people feel this is too verbose; this is part of what we pay for the added flexibility and functionality. Remember, you're free to mix and watch Has with any other validation library of your liking!

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
    'TooShort' => 'You need to be over 18 to ride this roller coaster.'
  ]));
}

```

Sometimes we'll want to do different things when certain errors are found in the passed data, so we can catch for specific exceptions. Now, this is tricky, because this error can be *within* the bigger exception being thrown, so we need to do this instead:

```php
$usernameValidator = Has::alphanumeric()->over(5)->under(20)->allow([' ']); // alphanumeric plus spaces

$usernameValidator->pushHandler('TooShort', function(TooShortException $e, ValidationException $v){
  log_that_someone_tried_inserting_too__short_a_username();
});

$usernameValidator->pushHandler('WhitelistViolation', function(WhitelistViolationException $e, ValidationException $v){
  log_someone_tried_a_username_thats_not_alphanumeric_with_spaces_those_monsters();
});

try {
  $usernameValidator->check("J-D"); // fails because it's too short and has a dash
} catch (ValidationException $e) {
  echo $e->getMessages();
}
```

Notice how those closures have a second argument? That's the end exception that gets thrown during the try-catch, so you can do things to the response if you'd like:

```php

$ageValidator = Has::integer()->over(18);

$ageValidator->pushHandler('range', function(TooShortException $e, ValidationException $v){
  $v->pushMessage('TooShort', "You're still under 18!");
});

try {
  $ageValidator->check(15);
} catch (ValidationException $e) {
  $messages = $e->getMessages();
}

// $messages would contain:
[
  'TooShort' => ["You're still under 18!", "Integer value 15 is under the minimum of 18"]
]

```

Notice that `pushMessage()` added the new message on top of the existing one, instead of replacing it. `replaceMessage()` would do that if you so fancy.

When doing "embedded" validation, these arrays will start nesting:

```php

$userJsonValidator = Has::key('user', Has::alphanumeric()->over(2)->under(50))
                        ->key('email', Has::email()->over(10)->under(50));
                        
try {
  $userJsonValidator->check(['user' => 'a', 'email' => 'e.com']); // would fail 3 validation checks
} catch (ValidationException $e) {
  $messages = $e->getMessages();
}      

// $messages would contain:
[
  'user' => [
    'TooShort' => "String 'a' is shorter than the minimum of 2"
  ],
  'email' => [
    'TooShort' => "String 'e.com' is shorter than the minimum of 10",
    'BadFormat' => "'e.com' is not a valid email address"
  ]
]

```

Formatting
==========

Where `$e->getMessages()`  returns a flat array, Has provides a number of formatters to turn that into a variety of formats. Do `$e->formatMessages(new Formatter)` to do this; `Formatter` should be one of the following:

JsonFormatter

```json
[
  'name' => [
    'TooShort' => "String '2' is shorter than the minimum of 2",
    'BadFormat' => "Your username should at least have one letter"
  ],
  'password' => [
    'Null' => "A password must be provided"
  ]
]

```


HtmlFormatter

```html
<ul>
  <li>
    <strong>Name</strong>
    <ul>
      <li class="TooShort">String '2' is shorter than the minimum of 2</li>
      <li class="BadFormat">Your username should at least have one letter</li>
    </ul>
  </li>
  <li>
    <strong>Password</strong>
    <ul>
      <li>A password must be provided</li>
    </ul>
  </li>
</ul>

```

I know what you're thinking. These formats look way too hardcoded - your HTML probably looks different. Feel free to implement your own Formatter - just implement the `Formatter` interface from Has, provide the `format(Array $messages)` method, and stick it into `formatMessages`!







