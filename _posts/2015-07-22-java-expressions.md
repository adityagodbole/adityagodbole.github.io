---
layout: post
title: Expressions in Java 8
---

Having worked in languages like Ruby, Coffeescript, Haskell, Scheme, etc. I miss expressions in other languages like Java.
Here is a simple way of writing expressions in Java 8, now that it has lambdas and functional interfaces

<!--more-->

In ruby, it is possible to write the following ...

```ruby
dir = if process.use_local_bin?
        "/usr/local/bin"
      else
        "/usr/bin"
      end
```

instead of

```ruby
dir = nil
if process.use_local_bin?
  dir = "/usr/local/bin"
else
  dir = "/usr/bin"
end
```
The former is an _expression_ while the later uses _statements_.
This may be a contrived example, but you see code like the above scattered
all over the place in a language like Java. For example,

```java
String dir;
if (process.useLocalBin())
  dir = "/usr/local/bin";
else
  dir = "usr/bin";
```

In Java this is of significance because it prevents us from making `dir` a **final** variable, a practice that is 
otherwised quite encouraged.
An expression like style can be achived quite trivially though, using lambdas and functional interfaces.
Add a small utility class like below -

```java
public class Let {
    public static <T> T let(Supplier<T> l) {
        return l.get();
    }
}
```
Then you can do -

```java
final String dir = let(() -> {
  if (process.useLocalBin())
    return "/usr/local/bin";
  else
    return "/usr/bin";
});
```

This variant of `let` does not accept any parameters. If required one can write overloaded variants of `let` corresponding to
the [functional interface options](https://docs.oracle.com/javase/8/docs/api/java/util/function/package-summary.html) provided in the standard library.
If these are not enough, it is trivial to construct interface for more elaborate types.
