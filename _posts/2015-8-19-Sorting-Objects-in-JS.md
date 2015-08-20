---
layout: post
title: Sorting Objects in JS
---

By way of showing what I intend to do here, let's talking about sorting associative arrays in JavaScript!

## The Problem
The problem is... there are no associative arrays in JavaScript. In other languages, there's often a way to assign keys in arrays, but JavaScript's arrays always use integer keys. Of course, JavaScript has objects, which can have almost any kind of key-value pairs you would want to use. But objects aren't ordered, so there's no strict concept of which key is first or last, and there's no method that tries to sort them.

But all the same, we can write one!

## Why Do We Need to Sort an Object?
Well, let's say we want to sort a table by a column. This is a common enough use case, and it's the perfect place for sorting an object.

__[Here's a fiddle!](http://jsfiddle.net/mistergone/dn6nz9u3/)__

## How Does It Work?
The first thing we do is handle the click event - we find the table, the row, the cell, and the index of the cell (which column was clicked), and we store all that. Then we find the value in that column in each row of the table, and as we do so, we fill an array, called `rows`. This array is __an array of arrays__. So, each value in `rows` is two arrays - the first is the value we got from the table cell, and the second is a jQuery object of the entire row (which will come in handy later).

## The Meaty Sort Function
The array of arrays, by itself, does not make sorting any easier - remember, each value in that array still has an integer key, and they have nothing to do with the values from the cells. But there's a neat thing about the JavaScript `Array.sort()` method - it can accept a function as a parameter, and that function call tell `Array.sort()` how to sort the Array.

Unfortunately, most of the solutions online don't seem to tell you about how to pass that function to `Array.sort()`.

For a simple numeric array (which `sort()` does by default), the function would end up looking like this...

```
someArray.sort( function(a, b) {
  # the parameters a and b are values in the Array which will be compared, so...
  if ( a < b ) {
    return -1;
    # Above, '-1' signifies that 'a' should be placed before 'b'
  }
  else if ( a > b ) {
    return 1;
    # Above, '1' signifies that 'b' should be placed before 'a'
  }
  else {
    return 0;
    # Above '0' signifies that 'b' and 'a' are equal
  }
}
```

This can be simplified for numbers because sort is actually only looking whether the returned value is positive, negative, or 0, so...

```
someArray.sort( function(a, b) {
  return a - b;
}
```

Of course, you can get a lot more complicated. If, for some reason, the number 5 should always be first, but otherwise the Array should be sorted numerically, this should work:

```
someArray.sort( function(a, b) {
  if ( a === 5 ) return -1;
  if ( b === 5 ) return 1;
  return a - b;
})
```

That looks silly, but it provides some insight into how the function works.

By knowing the ins and outs of `Array.sort( sortingFunction )`, you can sort complex objects by multiple dimensions (say, by last name, then first name) with more complex functions.

Well, I think it's neat. I hope you do, too.
