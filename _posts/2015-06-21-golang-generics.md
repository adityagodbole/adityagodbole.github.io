Date: 25 March 2015
Title: Golang generics via code generation using the C preprocessor

_This post is a followup to a talk I gave at the Pune golang meetup on using the C pre-processor to generate code to implement a poor man's generics. If you are familiar with the generics problem, feel free to jump to the cpp solution below._

When Go came out, it was positioned as a better C++. However, Go does not have generics. And interfaces are not generics, no matter what anyone says. So I don't see people who use C++ templates, moving to Go. And by that I don't mean that I don't think they won't move; I have asked a lot of people who write C++ and they have said they are not moving because they need generics.

After being heavily influenced by [SICP](http://mitpress.mit.edu/sicp/) and having used dynamically typed languages like Ruby and Javascript, and to a lesser extent Java 8 and Haskell for the last 5 odd years, I was quite disappointed that there was no simple way to express higher order process abstractions in Go. Sure it has first class functions. But in a statically typed language, they are not very useful without generics or algebraic data types. There are a couple of options to solve the generics problem though. The first one is to use reflection. Here are a couple of files that demonstrate how to implement a "generic" cons list using reflection.


<script src="http://gist-it.appspot.com/github/adityagodbole/golang-cpp-generation/blob/master/reflected.go?footer=minimal"></script>


<script src="http://gist-it.appspot.com/github/adityagodbole/golang-cpp-generation/blob/master/reflect_cons.go?footer=minimal"></script>

The above implementation has the following problems

* It is not typesafe
* Using `interface{}` is akin to using `void *` in C. You need to explicitly typecast the values
* There is a runtime hit. In an experiment involving creation of an actor using reflection, I got a 13% overhead compared to an implementation using a dispatch table.
* Implementing something as simple as a recursive sum is a PITA.

The other option is to implement something equivalent to generics using code generation. The options I came across were using the [ast package](http://golang.org/pkg/go/ast/) or using [go gen](http://clipperhouse.github.io/gen/). The AST option was just too low level for me. The go gen packages seems good, but there was no documentation regarding writing your own typewriters and overall I didn't think it was simple.

### The cpp solution

Initially I was attracted to Go because it is very similar to C in that it is simple, and has facilities that cater to fundamental orthogonal concepts of programming languages. So I had a hope that at least, Go could be a better C, at least where having a Go runtime is possible.

Recently, I happened to refer to some code I had written a long time ago in C and low and behold, I realised that I had solved the generics problem 10 years ago in C using code generation. In C, it's called macros! So I tried using the C pre-processor on go code interspersed with some C preprocessor (cpp) code. And it worked, more or less. One issue was that cpp removes all newlines, which is unacceptable in go. To get around this, I terminated all lines with `;\` instead of the usual `\` and wrote a script that calls the C preprocessor and pipes the result through a sed command that replaces `;` with newlines.

The biggest advantage of this approach is that a lot of programmers have experience with the C preprocessor and understand it. Also, the whole approach is very simple, almost naive.

Here is the result for the above cons list implementation using type specialised code generation.

#### The macro code

<script src="http://gist-it.appspot.com/github/adityagodbole/golang-cpp-generation/blob/master/pair.h?footer=minimal"></script>

#### The Go code that will pass through the pre-processor

<script src="http://gist-it.appspot.com/github/adityagodbole/golang-cpp-generation/blob/master/pair.go.H?footer=minimal"></script>

#### The Go driver code

<script src="http://gist-it.appspot.com/github/adityagodbole/golang-cpp-generation/blob/master/generated.go?footer=minimal"></script>

#### The gopp (go pre-processor) script

<script src="http://gist-it.appspot.com/github/adityagodbole/golang-cpp-generation/blob/master/gopp?footer=minimal"></script>

### A few notes

* Only files ending with `.go.H` get acted upon by the gopp script. The code generated for a file named `foo.go.H` will be `foo_gen.go`.
* The generic type that is generated is named as `Foo_T` where `T` is the type for which the code is specialised. Eg. if in Java one would write `Cons<int>`, here it would be `Cons_int`.
* This approach doesn't really solve the problem of creating libraries that implement generic types or algorithms (unless they are distributed as `.h` files), but it is a decent start
* The full reference code can be found [on github](https://github.com/adityagodbole/golang-cpp-generation)
