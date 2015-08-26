---
layout: post
title: Fun with String.replace()
---

It's time again for another fun exploration of the depths of Javascript. Did you know you could pass a function to the `String.replace()` method? It can make some complex [regex](https://en.wikipedia.org/wiki/Regular_expression) situations much easier.

## The Setup
In my case, the setup was attempting to make a string into a number. `parseInt()` does this, but with complications. Try this in your local JS console:
```
> parseInt('99,999');

99
```
Wait, what? Yep, `parseInt()` will just ignore numbers after the first non-numeric character it finds. This is basically useless for parsing user input - users will type commas, or accidentally type letters, or two decimal points, and they will expect your app to handle it without erasing most of what they wrote.

## The Solution
There are a lot of solutions to this problem. Mine ends up being two regular expressions using the `String.replace()` method.

The first `replace()` is probably familiar to some people - replace all non-numeric characters with an empty String:
```
numberString = numberString.replace( /[^0-9\.]+/g, '' );
```
Simple, right? This will fix all our problems. Well, I hinted above at one potential problem here - we're ignoring decimal points, because they're necessary to properly parse a number, right? After all, we want '1,234.56' to parse as '1234.56', right?

Once again, let's try this in our JS console:
```
> numberString = '5.5.5';
> numberString = numberString.replace( /[^0-9\.]+/g, '' );

"5.5.5"
```
Oh. That's not going to work. There are two decimal points in that string, which means when we try to turn it into a Number...

```
> Number('5.5.5');

NaN
```
Well, we never want to see that. So let's do this - let's assume the user meant the first decimal point only, and all other decimal points should be ignored. How do we make a regex for that? Well, in JavaScript, it's actually a little difficult because JS's implementation of regular expressions doesn't have this thing called [lookbehind](http://www.regular-expressions.info/lookaround.html). But don't worry about that - regular expressions are complicated.

Instead, let's check out passing a function to `String.replace()`.

## The Complexities of String.replace();

This is how a simple `String.replace()` looks:
```
> 'I hate dogs.'.replace( 'hate', 'love');

"I love dogs."
```

Instead, we can use a regular expression as the first parameter to match:
```
>'I hate dogs.'.replace( /hate/g, 'love');

"I love dogs."
```

But we can also pass a function as the second parameter, which we can use to perform tests against the match and return whatever we want to replace the match with. That takes this form:
```
> 'a,b,c,d,e'.replace( /[a-z]/g, function( match, offset, full ) { 
      return '(The letter ' + match + ' was found at position ' + offset + '.)\n'; 
    });

"(The letter a was found at position 0.)
,(The letter b was found at position 2.)
,(The letter c was found at position 4.)
,(The letter d was found at position 6.)
,(The letter e was found at position 8.)
"
```

In the above example, three parameters are passed to the function:
 - `match` is the character(s) matched
 - `offset` is the offset of those characters in the string, starting at 0, of course
 - `full` is the full string, which can be extremely useful

__WARNING!__ The parameters passed to the function inside `replace()` __might be different__ depending on the regular expression passed as the first parameter to `.replace()`. For more information on what that means, you can check [the Mozilla reference to the method](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/replace#Specifying_a_function_as_a_parameter).

Oh yeah, sorry about the warning, but I just wanted to make sure you know about that. Anyway, this function is extremely useful. Now, we can pass each match to a function. To solve our original problem, turning `'5.5.5'` in `'5.55'`, what I'll do is compare `offset` to `full.indexOf( '.' )`. If they're equal (meaning it's the first decimal point in the string), I'll "replace" the decimal point with a decimal point. Otherwise, I'll replace it with an empty String!

In this example, I'll also declare the function outside of `String.replace()` to make it much easier to read.
```
function replaceCommas( match, offset, full ) {
  if ( offset === full.indexOf( '.' ) ) {
    return '.';
  }
  return '';
}
'5.5.5'.replace( /\./g, replaceCommas );
```
Put that in our JS console, and we get...
```
5.55
```

Alright, problem solved! Now, our whole input parsing function looks something like this:
```
// Strip non-numeric values, maintaining periods
numberString = numberString.replace( /[^0-9\.]+/g, '' );

// Strip any periods after the first
function replaceCommas( match, offset, full ) {
  if ( offset === full.indexOf( '.' ) ) {
    return '.';
  }
  return '';
}
numberString = numberString.replace( /\./g, replaceCommas );

return Number( numberString );
```
If you're particularly observant, you might notice the code doesn't handle negative numbers. There are a lot of ways to do that, too, but that's for another post.

## More Applications
Using a function in `String.replace()` makes handling user input a lot easier. You could imagine a very complex function passed to `String.replace()` that actually matches every individual character of a String, performs a series of tests on it, even using `match`, `offset`, and `full` to see how it relates to its neighbors and the String as a whole, then returning some permutation of the original character. Instead of a series of checks and `String.replace()` calls, you can do it all in one function with one call of `String.replace()` on the String.

Hope this was helpful!
