---
layout: post
title: To RegEx Or Not To RegEx, That Is The /\?/
date: 2016-01-05 20:00:39
categories: RegEx
---
Regular expressions are a great way to solve some difficult questions, but often faster and easier to read alternatives exist.

Regular expressions can even start a good argument. One of the most up voted answers on stackoverflow is in response to a 

>[Regex Match Open Tags...](http://stackoverflow.com/questions/1732348/regex-match-open-tags-except-xhtml-self-contained-tags/1732454#1732454)
You can't parse [X]HTML with regex. Because HTML can't be parsed by regex. Regex is not a tool that can be used to correctly parse HTML. As I have answered in HTML-and-regex questions here so many times before, the use of regex will not allow you to consume HTML. Regular expressions are a tool that is insufficiently sophisticated to understand the constructs employed by HTML.

The answer continues.

>...Every time you attempt to parse HTML with regular expressions, the unholy child weeps the blood of virgins, and Russian hackers pwn your webapp... ...un̨ho͞ly radiańcé destro҉ying all enli̍̈́̂̈́ghtenment, HTML tags lea͠ki̧n͘g fr̶ǫm ̡yo​͟ur eye͢s̸ ̛l̕ik͏e liq​uid...

Lets look at a few situations where regular expressions could be used to solve problems and then compare to alternatives.

To remove subjectivity, I'll run timed test and then explain to my [rubber ducky](http://en.wikipedia.org/wiki/Rubber_duck_debugging) what each function is doing. If my rubber duck understands one way over the other it will be considered more readable.

# Form Validation
___
A common place to find regex is inside validating functions. Lets say we need to validate a password before it is saved (salted and hashed of course). Lets say we want to make sure that the password is 7 - 20 characters long and has at least one upper case, one lower case and one number. Also since we strive for good UX, we will be checking on the client side (JavaScript) and then once again on the server (PHP). Oh yeah I almost forgot, no whitespace.

All functions should return true for valid and false for invalid.

Here is what our js test may look like.
___
<!-- ' -->
{% highlight js %}
// Using regex
function isPasswordValidRegex(password) {
  var regex = /^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])(?!.*[\s]).{7,20}$/; // My rubber ducky is confused
  return password.match(regex) !== null;
}

// without regex
// this should be interesting
function isPasswordValidNoRegex(password) {
  var len = password.length;
  var hasDigit = false;
  var hasUpperCase = false;
  var hasLowerCase = false;

  // check length - easy
  if (len < 7 || len > 20) {
    return false;
  }

  // check if contains space - easy
  if (password.indexOf(' ') !== -1) {
    return false;
  }

  // verify password contains one upper one lower and a digit - not so easy
  if (this.hasDigit(password) && this.hasLowerCase(password) && this.hasUpperCase(password)) {
    return true;
  }
  return false;
}
{% endhighlight %}

hasDigit, hasLowerCase, hasUpperCase
___
{% highlight js %}
// this was my first thought to check for lowercase
// but it will not work for unless you remove all number special characters, etc.
function hasLowerCase(password) {
  var i, len; 
  for (i = 0, len = password.length; i < len; i++) {
    if (password.charAt(i) === password.charAt(i).toLowerCase()) {
      return true;
    }
  }
  return false;
}

// This will work though
function hasLowerCase(password) {
  var i, len;
  for (i = 0, len = password.length; i < len; i++) {
    if (password.charAt(i) >= 'a' && password.charAt(i) <= 'z') {
      return true;
    }
  }
}

function hasUpperCase(password) {
  var i, len;
  for (i = 0, len = password.length; i < len; i++) {
    if (password.charAt(i) >= 'a' && password.charAt(i) <= 'z') {
      return true;
    }
  }
}

function hasDigit(password) {
  var i, len;
  for (i = 0, len = password.length; i < len; i++) {
    if (!isNaN(parseFloat(password.charAt(i)))) {
      return true;
    }
  }
  return false;
}
{% endhighlight %}

Ok wow that was a lot of work. I think if I was only validating passwords the complexity of the second choice would make me use a regex.
I had a really hard time explaining `RegExp(/^(?=.*\d)(?=.*[a-z])(?=.*[A-Z])(?!.*[\s]).{7,20}$/, '')` to my rubber duck, but he got bored, fell asleep and rolled off the desk before I could explain the second method to him.
In anycase lets see if we gain a speed advantage for our trouble.

{% highlight js %}
function testSpeed(iterations, func, password) {
  var i, end, begin;
  begin = Performance.now();
  for (i=0; i < iterations; i++) {
    func(password);
  }
  end = Performance.now();
  console.log(end - begin);
}

{% endhighlight %}

## Results
___
testSpeed(1000000, isPasswordValidRegex, "invalidstring");
115.02999999979511

testSpeed(1000000, isPasswordValidRegex, "ISvalidstr1ng");
280.625

testSpeed(1000000, isPasswordValidNoRegex, "invalidstring");
1097.7649999998976

testSpeed(1000000, isPasswordValidNoRegex, "ISvalidstr1ng");
1275.0749999999534

`Version 47.0.2526.106 (64-bit)`

Ok looks like regex wins all the way around her.

Lets try with a simpler validation. Just check if it is 7 - 20 characters long.
{% highlight js %}
function isPasswordValidRegex(password) {
  var regex = /^.{7,20}$/; // My rubber ducky is confused
  return password.match(regex) !== null;
}

function isPasswordValidNoRegex(password) {
  var len = password.length;
  if (len < 7 || len > 20) {
    return false;
  }
  return true;
}
{% endhighlight %}

## Results
___
testSpeed(1000000, isPasswordValidRegex, "ISvalidstr1ng");
137.27000000001863

testSpeed(1000000, isPasswordValidNoRegex, "ISvalidstr1ng");
8.965000000083819

Interesting results! I would call this a win for no regex both for simplicity and speed.

My conclusion is the same as what I was told by the experts around me, use regex if you need it, but don't use it for simple cases. Looks like performance will not matter either way unless you are checking thousands of records.