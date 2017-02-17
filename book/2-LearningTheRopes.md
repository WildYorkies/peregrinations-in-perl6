# In Which I Show You the Ropes

## Syntax

## Interesting idioms

### gather/take

instead of loop push, use gather/take

### multiple dispatch

instead of one function with a ton of if/else logic, write multiple
functions that take different parameters

### ranges

^$number

1..$number

1,3,5, ... ^100

## Objects

## Reading Documentation

I'm going to throw you into the deep end with Perl6, so you'll need a companion on this
journey. That companion will be [the docs](docs.perl6.org). Whenever you come across something you don't
understand, go to the docs. In fact, just keep that page open while reading this.

### How to find what you're looking for

You can either search for something directly in the docs, or browse the docs and try to find it
that way.

#### Browsing 

The docs have some main subsections: Types, Language, etc 

Perl6 is object oriented at its core. So, built in types (objects) and methods will be in the
Types area of the docs. This is where you will spend most of your time. 

Go ahead and click Types and then click on the Str type. Browse the table of contents and you
will see the methods that the Str type responds to, then you'll also see the methods that
the Str type inherets from parent types or roles. 

Find the `comb` method for the Str type.

### How to read what you're looking at 

For `comb`, you should see something like this:

```
stuff
```

The first thing you may be wondering is, what's the difference between a routine and a method? Not much.

A method call is `$object.do-this($some-arg)`. If that same method had a "routine" version of itself
it would look like this `do-this($object, $some-arg)`.

So you should notice that comb has a routine version and a method version. In reality, the routine version
is syntax sugar for the method version.

Digging into the docs for `comb`, let's look at the list of signatures for the routine and method.

```perl6
stuff
```

`method comb(Str:D, )`

This signature for the comb method means that it operates on a defined Str object. Objects can be defined
or undefined Perl6. You could hypothetically have a version of comb that operates on an undefined Str object
and the docs for that might look like `method comb(Str:U, )`

Note that even though these signatures in the docs have `Str:D` inside the parens, it is still a normal
method call like `$my-defined-str.comb`.

