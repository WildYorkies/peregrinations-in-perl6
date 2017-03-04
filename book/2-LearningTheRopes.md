# In Which I Show You the Ropes

Firstly, read through http://perl6intro.com 

When you've fully browsed that site, come back.

## Idiomatifyer 9000

In this section, we're going to show some small programs and continue to implement Perl 6 idioms until
it is properly optimized with awesomeness. The goal is to simply get you acquainted with some of these
features of the language.

### 

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
routine comb
```
```perl6
multi sub    comb(Str:D   $matcher, Str:D $input, $limit = Inf)
multi sub    comb(Regex:D $matcher, Str:D $input, $limit = Inf, Bool :$match)
multi sub    comb(Int:D $size, Str:D $input, $limit = Inf)
multi method comb(Str:D $input:)
multi method comb(Str:D $input: Str:D   $matcher, $limit = Inf)
multi method comb(Str:D $input: Regex:D $matcher, $limit = Inf, Bool :$match)
multi method comb(Str:D $input: Int:D $size, $limit = Inf)
```
```
Searches for $matcher in $input and returns a list of all matches (as Str by default, or as Match if $match is True), limited to at most $limit matches.

If no matcher is supplied, a list of characters in the string (e.g. $matcher = rx/./) is returned.
```

The first thing you may be wondering is, what's the difference between a routine and a method? Not much.

A method call is `$object.do-this($some-arg)`. If that same method had a "routine" version of itself
it would look like this `do-this($object, $some-arg)`.

So you should notice that comb has a routine version and a method version. In reality, the routine version
is syntax sugar for the method version.

Digging into the docs for `comb`, let's look at the list of signatures for the routine and method.

```perl6
multi sub    comb(Str:D   $matcher, Str:D $input, $limit = Inf)
multi sub    comb(Regex:D $matcher, Str:D $input, $limit = Inf, Bool :$match)
multi sub    comb(Int:D $size, Str:D $input, $limit = Inf)
multi method comb(Str:D $input:)
multi method comb(Str:D $input: Str:D   $matcher, $limit = Inf)
multi method comb(Str:D $input: Regex:D $matcher, $limit = Inf, Bool :$match)
multi method comb(Str:D $input: Int:D $size, $limit = Inf)
```

Specifically look at the methods like `method comb(Str:D, )`.

This signature for the comb method means that it operates on a **defined** Str object. Objects can be defined
or undefined Perl6. You could hypothetically have a version of comb that operates on an undefined Str object
and the docs for that might look like `method comb(Str:U, )`

Note that even though these signatures in the docs have `Str:D` inside the parens, it is still a normal
method call like `$my-defined-str.comb`.

Now scroll down a bit and you can read some explanation of how the routine works, but the best thing about
Perl6 docs are the _examples_.

```perl6
say "abc".comb.perl;                 # OUTPUT: «("a", "b", "c").Seq␤» 
say 'abcdefghijk'.comb(3).perl;      # OUTPUT: «("abc", "def", "ghi", "jk").Seq␤» 
say 'abcdefghijk'.comb(3, 2).perl;   # OUTPUT: «("abc", "def").Seq␤» 
say comb(/\w/, "a;b;c").perl;        # OUTPUT: «("a", "b", "c").Seq␤» 
say comb(/\N/, "a;b;c").perl;        # OUTPUT: «("a", ";", "b", ";", "c").Seq␤» 
say comb(/\w/, "a;b;c", 2).perl;     # OUTPUT: «("a", "b").Seq␤» 
say comb(/\w\;\w/, "a;b;c", 2).perl; # OUTPUT: «("a;b",).Seq␤» 
```

Take some time to try these examples in the Perl6 repl on your command line or terminal.
