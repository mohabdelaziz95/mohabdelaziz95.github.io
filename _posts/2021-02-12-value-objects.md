---
title: "Value Objects - Your Savior from Primitive Obsession"
description: "Save your codebase from naive primitive types — no more (int, string, etc..) but meaningful types."
date: 2021-02-12 00:00:00 +0200
# Social-card image: the exact URL already shared since 2021 — keep this path.
image: /assets/images/posts/4334233.jpg
hero: /assets/images/posts/value-objects-hero.webp
hero_width: 1600
hero_height: 1064
hero_alt: "A still from the film The Last Samurai"
hero_caption: 'From the movie <a href="https://www.imdb.com/title/tt0325710/mediaviewer/rm61508864/" target="_blank" rel="noopener">The Last Samurai</a> on <a href="https://www.imdb.com/title/tt0325710/" target="_blank" rel="noopener">IMDB</a>'
card_image: /assets/images/posts/value-objects-card.webp
keywords: "Value Objects, Primitive Obsession"
---

I believe that the best way to explain a problem is to introduce the problem itself first. I know that sounds unquestionable, but people always forget that. They jump to the solution without figuring out what the actual problem is — we all do that.

## What is the problem?

Here we have an example; imagine you are building an eCommerce platform which allows users to sell anything and you are offering them a shipping option, and to determine if this item is deliverable or not you have this `ShippingRequest` class.

```php
class ShippingRequest {

   /**
    * @param mixed $isDeliverable
    *      - false: no need for shipping service
    *      - true:  ship my product
    *      - null:  if to be decided
    */
   public function checkAvailabilityForDelivery($isDeliverable)
   {
        //...
   }
}
```

And since the shipping service is optional, we are allowing our sellers — if they want to ship their product just select the delivery flag. If not, select the not-deliverable flag, or just leave it empty if you didn't decide yet. And on your side (backend) based on this value you have this `checkAvailabilityForDelivery` which accepts an `$isDeliverable` argument, and based on this argument value you need to take a decision, and this value might be true, false, null.

And this will be your lucky day if you even got the `TRUE`, `FALSE` as bool types — but you forgot that you are sending this value over an HTTP request, where basically everything is a STRING, so the TRUE will be `"TRUE"` and FALSE will be `"FALSE"`. And if you are working with a language like PHP this will not be your last problem. Yes, `"false"` is considered as true in PHP, as PHP translates that to a non-empty string value, so it's simply TRUE.

```php
var_dump((bool) "false");     // true
var_dump("false" == false);   // false
```

Now I hear the voice in your head saying what the hell is this?

![Jackie Chan looking confused](/assets/images/posts/wtf.webp){: width="820" height="488" loading="lazy" decoding="async"}

You may say now, if the problem is that I am accepting a `mixed` type then simply I'll not use it. I'll just accept only one type.

Let's check this other example: you are sending a welcome email to the newly registered users of your eCommerce app using this `WelcomeEmail` class, through the `sendEmail` method.

```php
class WelcomeEmail {

    /**
    * @param string $emailAddress
    * @param string $subject
    * @param string $body
    */
   public function sendEmail(string $emailAddress, string $subject, string $body)
   {
        //...
   }
}
```

Now you are expecting a `string $emailAddress`, `string $subject`, `string $body`. Everything looks good, so let's use it.

```php
(new WelcomeEmail())->sendEmail(
       "registration success",
       "glad to have u with us :)",
       "johndoe"
);
```

## You have an Issue!

The expected behavior: when a new user registers to your platform, that user should receive a welcome email.

But the actual behavior: the user registered, but they didn't receive any emails.

"If you think that you can rely on a developer's attention…"

![An animation of someone losing their patience](/assets/images/posts/attention.webp){: width="498" height="339" loading="lazy" decoding="async"}

So instead of having these meaningless generic arguments with string values, now we have valid meaningful arguments by introducing a new type that has a meaning which goes beyond being only a primitive type string.

```php
final class WelcomeEmail {

    /**
    * @param EmailAddress $emailAddress
    * @param Subject $subject
    * @param Body $body
    */
   public function sendEmail(EmailAddress $emailAddress, Subject $subject, Body $body)
   {
        //...
   }
}
```

This also helps you avoid mixing parameters, since the first parameter expects a type of EmailAddress, not just a string. We call those new types **ValueObjects**.

## What is a ValueObject?

> An object that represents a descriptive aspect of the domain with no conceptual identity is called a Value Object. Value Objects are instantiated to represent elements of the design that we care about only for what they are, not who or which they are.
>
> <cite>— Eric Evans</cite>

### Which means?

1. A value object is an object that is defined according to its value or its data rather than its identity.
2. A value object represents a typed value in your domain.
3. Also, this means we are representing things that are related to each other as a compound object or type.

For example:

- A 2D coordinate consists of an X value and a Y value.
- An amount of money consists of {number + currency}.
- A date range consists of {`start_date` + `end_date`}.

Let's say that you have a method to calculate the distance between two points, and each point has a pair of (x, y) values, so it might look like this:

```php
function calculateDistance(Coordinates $startPoint, Coordinates $endPoint)
{
     //...
}
```

Let's take another example:

```php
function register($age)
{
    dump("your age is {$age}");
}
```

This is a simple register function responsible for registering new users to your platform. So for simplicity, let's have it accept only one argument which is `$age`, and then you can just call this function and give it the user's age, which is in our case 10.

```php
register(10);
```

Then you got a new issue reported: that your `register` function accepts an age that is zero or less than zero, or greater than 200, which does not make any sense.

So instead of that, let's introduce a new type in our codebase called `Age`, which is always responsible for giving you a valid age whenever needed.

```php
final class Age {

    public $age;

    public function __construct($age)
    {
        if ($age < 0 || $age > 120) {
            throw new InvalidArgumentException("provided age is invalid");
        }

        $this->age = $age;
    }

    public function __toString()
    {
        return $this->age;
    }
}
```

Then let's modify the register function signature, and instead of defining that this function accepts a meaningless `$age` argument, it accepts a valid age value of type `Age` — and if the user provided an invalid age value, it will throw an `InvalidArgumentException` saying that the provided age value is invalid.

## ValueObject characteristics

### 1. No identity

- ValueObjects are defined by their attributes or data. They are equal if their attributes are equal, and they are completely interchangeable.
- Unlike entities, they don't have an ID property.

#### So what is an identity?

- An object that represents a specific thing would be an entity. E.g. Models → represent Entities.
- ValueObjects are simple objects that represent simple things, but not specific things.

E.g. imagine you have 5 banknotes of \$10 and you asked someone to take a \$10 note — does it really matter which banknote they took? Isn't the important thing that they took a \$10 banknote? But if we said that the police are looking for a missing \$10 banknote which has a serial number of 0xxxxxxx0, now this banknote has an identity.

- A ValueObject can't exist without a parent Entity owning them. E.g. saying an address like this (Egypt, Cairo, street number, building number) doesn't provide a full comprehensive statement. But if we mention that John Doe's address is (Egypt, Cairo, street_number, building_number), now this makes complete sense.
- ValueObjects should not have separate tables in the DB.
- There always must be a composition relationship between a ValueObject class and an Entity class; without it ValueObjects don't make any sense.

### 2. Immutability

- **Mutable object**: an object whose internal state can be changed.
- **Immutable object**: an object whose internal state cannot be changed.
- The only way to change its value is by a full replacement (a new instance with a new value).
- **Immutable** → NO SETTERS, NO PUBLIC PROPERTIES; the constructor is the only injection point.

E.g. in our previous age example, we defined the `$age` property in the `Age` class as public. And this means that any user of the Age type can easily change the value — so instead of that, we need to change it to be `private` so no one can access or modify the age value.

```php
$age = new Age(10);
$age->age = 1000;
register($age);   // 1000
```

### 3. Self validation

- A ValueObject must verify the validity of its attributes when being created.
- An error or exception should be raised if any of the attributes are invalid.

## Reasons for ValueObjects to exist in your codebase

### 1. Reduce primitive obsession

Primitive obsession is using primitive data types {int, string, float, etc...} to represent domain ideas.

E.g. we usually store a **URL** as a String, but a **URL** has more information and specific properties compared to a String — and by storing it as a string you can no longer access these **URL**-specific properties (the domain concept) without additional code.

```
              Host          Port                       Query Params
        ┌──────┴──────┐ ┌┴┐                    ┌────────────┴───────────┐
https://www.example.com:123/forum/questions/?tag=networking&order=newest
└─┬─┘                      └───────┬───────┘
Protocol                          Path
```

### 2. Type safety

When using primitive types, it is easy to make mistakes and bugs even when we are using valid types but we are passing invalid values.

E.g. `private float $duration;`

Float here is a valid type, and when you initialize this variable with a float value your compiler/interpreter won't complain about this, since you're giving it a valid float value. But what about the type of the `Duration` itself, since the `Duration` could be (seconds, minutes, hours, etc..)?

You might now scratch your head and say why not just rename the variable — you can just mention it explicitly that you want the duration in seconds, so it might be something like `$durationInSeconds`.

What if someone from your business team requested a new change request, as they always do? They want to display the duration in minutes and hours, so now you need to convert the duration to adopt the new changes, which means extra conversion code. So you might ask yourself about your options: where should this conversion code reside?

- Put it inline just right before the usage! What if we decided that we need to use Duration in minutes somewhere else? No copy/paste. Don't Repeat Yourself.
- You might put it in as a helper function in a helpers file where all the evils reside. I think you need to reconsider this decision again.

Let's go one step back and ask ourselves what we really need here. We need to ensure that when we always ask for a valid duration we get a valid duration. When we ask for seconds, we get only seconds. Ask for minutes and we should get only minutes, etc…

We also need to ensure our conversion code is consistent in all places in our codebase. We actually need a ValueObject to represent the Duration type in our domain, which will contain all the validation logic and the conversion logic.

```php
final class Duration {

    private $seconds;

    public function __construct(int $seconds)
    {
        $this->seconds = $seconds;
    }

    public function asMinutes(): int
    {
        return (int) floor($this->seconds / 60);
    }
}
```

### 3. Mixing parameters

Imagine if you're using an IDE that does not support showing parameter name hints, or you are just editing something in VIM — you can easily end up mixing valid parameters of the same type.

```php
function sendShippingData(string $courierName, string $trackingID)
{
    //...
}

shippingData("#123456", "DHL");
```

```php
final class ShippingInstrument {

    private $trackingID;

    private $courierName;

    public function __construct(string $courierName, TrackingID $trackingID)
    {
        if ($trackingID === "") {
            throw new InvalidArgumentException("Invalid shipping data");
        }

        $this->trackingID = $trackingID;
        $this->courierName = $courierName;
    }

    public function getCourier()
    {
        return CourierStorage::where('name', $this->courierName)->first();
    }
}
```

You get the idea now.

### 4. Avoid revalidating

With value objects you will stop revalidating data everywhere. You just validate it once, and after that you should be able to use it anywhere with confidence and a guarantee that it is valid. This is because the only way for a value object to exist is to be valid; once it exists you cannot change it. Immutable, do you remember?

### 5. Easy to read

Using value objects you don't have to guess what variables truly are. You no longer have to worry about internal representation, but think about domain concepts instead — and the code now looks a lot more expressive.

### 6. Ensure consistency

Check the duration example above.

## Drawbacks

- Wrapping every single primitive type you have with a ValueObject leads to creating too many classes, and this will bloat the codebase, leading to a ridiculous codebase that you don't want to work with. So choose wisely what might be a good ValueObject candidate.
- Performance issues: in some cases creating too many instances might lead to a performance drop.

## Load your brain with ValueObjects

At the beginning it might be hard to recognize ValueObject candidates, but with the passage of time it becomes easier. Moreover, I found some useful tricks that you might follow:

- You have a special validation logic that has to be applied in multiple places. E.g. phone numbers, email addresses, etc…
- You have a special formatting logic that is used to render values for humans. E.g. Price = Amount + Currency.
- In general, if you find yourself in a situation where you are constantly passing variables together, then this might be a good candidate for a ValueObject.

## References

- [ValueObject — Martin Fowler](https://martinfowler.com/bliki/ValueObject.html)
- [Value Objects explained](https://enterprisecraftsmanship.com/posts/value-objects-explained/)
- [Value Objects and Mutability — Laracasts](https://laracasts.com/series/object-oriented-principles-in-php/episodes/8)
- [Value Objects to the rescue!](https://medium.com/swlh/value-objects-to-the-rescue-28c563ad97c6)
