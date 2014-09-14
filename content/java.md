---
title: All the Java You Need
link_title: All the Java You Need
kind: documentation
draft: true
---

# All the Java You Need

There comes a day in every Clojurist's life when she must venture
forth from the sanctuary of pure functions and immutable data
structures into the wild and barbaric Land of Java. This treacherous
journey is necessary because Clojure is hosted on the Java Virtual
Machine (JVM), granting it two fundamental characteristics. First, you
run Clojure applications the same way you run Java
applications. Second, you need to use Java objects for core
functionality like reading files and working with dates. In this way,
Clojure is a bit like a utopian community plunked down in the middle
of a non-utopian country. It's preferable to interact with other
utopians, but every once in a while you need to talk to the locals in
order to get things done.

If this chapter were a guide book for the Land of Java, it'd be
somewhere between a phrasebook ("Where is the bathroom?") and a
full-blown course on the local language and customs. It will give you
an overview of what the JVM is, how it runs programs, and how to
compile programs for it. It will also give you a tour of
frequently-used Java classes and methods and explain how to interact
with them from Clojure. More than that, it will show you how to think
about and understand Java so that you can incorporate any Java library
into your Clojure program.

# The JVM

Developers use the term "JVM" to refer to a few different
things. You'll hear them say, "Clojure runs on *the* JVM", and you'll
also hear "Clojure programs run in *a* JVM". In the first case, "JVM"
refers to an abstraction - the general movel of the Java Virtual
Machine. In the second, it refers to a process, an instance of a
running program.  Right now, we're only concerned with the JVM model;
I'll point out when we're talking about running JVM processes.

To understand the Java Virtual Machine, let's first take a step back
and review how plain-ol' machines (also known as computers) work. Deep
in the cockles of a computer's heart is its CPU, and the CPU's job is
to execute operations like "add" and "unsigned multiply". These
operations are represented in assembly language by mnemonics like
"ADD" and "MUL". What operations are availabe depends on the CPU
architecture (x86, ARMv7 and what have you) as part of the
architecture's *instruction set*.

Because it's no fun to program in assembly language, we humans have
invented higher-level languages like C and C++, which get compiled
into the instructions that a CPU will understand. Here's a diagram of
the process:

1. Compiler reads source code.
2. Compiler outputs a file containing CPU instructions
3. The computer executes those instructions

The most important thing here is that, ultimately, you have to
translate programs into the instructions that a CPU will understand,
and the CPU doesn't care what programming language was used to produce
the instructions.

The JVM is analagous to a CPU in that it also executes low-level
instructions, called Java bytecode. As a *virtual* machine, though,
it's implemented as software rather than hardware. A running JVM
executes bytecode by translating it on the fly into the machine code
that its host will understand, a process called just-in-time
compilation.

For a program to run on the JVM, then, it has to get compiled to Java
bytecode. Usually, when you compile programs you store the result as a
JAR (Java archive) file. Just as a CPU doesn't care how machine
instructions are generated, the JVM doesn't care how bytecode gets
created. It doesn't care if you used Scala, JRuby, Clojure, or even
Java to create Java bytecode. Here's a diagram:

1. Compile to bytecode
2. JVM reads bytecode
3. JVM sends machine instructions

So when someone says that Clojure runs on the JVM, one of the things
they mean is that Clojure programs get compiled to Java bytecode and
JVM processes execute them. This matters because you treat Clojure
programs the same as Java programs from an operations perspective. You
compile them to JAR files and run them using the `java` command. If a
client needs a program that runs on the JVM, you could secretly write
it in Clojure instead of Java and they would be none the wiser. Seen
from the outside, you can't tell the difference between a Java and a
Clojure program any more than you can tell the difference between a C
and a C++ program. Clojure allows you to be productive *and* sneaky.

# Compiling and Running a Java Program

I think it's time to see how all of this works with real code. In this
section, you'll build a simple pirate phrasebook using Java. This will
help you feel much more comfortable with the JVM, it will prepare you
for the upcoming section on Java interop, and it will prove invaluable
should a scallywag ever attempt to scuttle your booty on the high
seas. To tie it all together, you'll take a peek at some of Clojure's
Java implementation.

## Hello World

Go ahead and create new directory called "phrasebook". Within that
directory, create a file named `PiratePhrases.java`, and write the
following in it:

```java
public class PiratePhrases
{
    public static void main(String[] args)
    {
        System.out.println("Shiver me timbers!!!");
    }
}
```

This defines a very simple program which will print the phrase "Shiver
me timbers!!!" (which is how pirates say "Hello, world!") to your
terminal when you run it. Now, in your terminal, compile this source
code with the command `javac PiratePhrases.java`. If you typed
everything correctly *and* you're pure of heart, you should see a file
named `PiratePhrases.class`:

```bash
$ ls
PiratePhrases.class PiratePhrases.java
```

You've just compiled your first Java program, son! Now run it with
`java PiratePhrases`. You should see:

```
Shiver me timbers!!!
```

What's happening here is you used the Java compiler, `javac`, to
create a Java class file, `PiratePhrases.class`. This file is packed
with oodles (well, for program this size, maybe only one oodle) of
Java bytecode.

When you ran `java PiratePhrases`, the JVM first looked on your
*classpath* for a class named `PiratePhrases`. You can think of the
classpath as the list of filesystem paths that the JVM will search in
order to find a file which defines a class. By default, the classpath
includes the directory you're in when you run `java`. Try running
`java -classpath /tmp PiratePhrases` and you will get an error, even
though `PiratePhrases.class` is right there in your current directory.

In Java, you're only allowed to have one public class per file and the
filename and class name must be the same. This is how `java` knows to
try looking in `PiratePhrases.class` for the ShiverMeTimbers class's
bytecode. After `java` found the bytecode for the `PiratePhrases`
class, it executed that class's `main` method. Java's kind of like C
that way, in that whenever you say "run something, and use this class
as your entry point", it always will run that class's `main`
method. Which means that that method has to be `public`, as you can
see above.

In the next section you'll learn about handling program code that's
spread over more than one file. If you don't remove your socks now,
they're liable to get knocked off!

## Packages and Imports

In this section, you'll learn about how Java handles programs which
are spread over more than one file. You'll also learn how to use Java
libraries. Once again, we'll look at both compiling and running a
program. This section has direct implications for Clojure, where
you'll the same ideas and terminology to interact with Java libraries.

First, a couple definitions:

* **package:** Similar to Clojure's namespaces, packages provide code
  organization. The directory that a Java file lives in must mirror
  the package it belongs to. If a file has the line `package
  com.shapemaster` in it, then it must be located at `com/shapemaster`
  somewhere on your classpath.
* **import:** Java allows you to import classes, which basically means
  that you can refer to them without using their namespace prefix. So,
  if you have a class in `com.shapemaster` named `Square`, you could
  write `import com.shapemaster.Square;` or `import com.shapemaster.*;`
  at the top of a `.java` file so that you can use `Square` in your
  code instead of `com.shapemaster.Square`. You'll see this
  illustrated below.

Now it's time to try out `package` and `import`. To start, you'll
create three files. First, create PirateConversation.java:

```java
import pirate_phrases.*;

public class PirateConversation
{
    public static void main(String[] args)
    {
        Greetings greetings = new Greetings();
        greetings.hello();

        Farewells farewells = new Farewells();
        farewells.goodbye();
    }
}
```

The first line, `import pirate_phrases.*;`, imports all classes in the
`pirate_phrases` package, which will contain the `Greetings` and
`Farewells` classes. Let's create those now. First, create the
directory `pirate_phrases`. This is needed because Java package names
correspond to filesystem directories. Then, create `Greetings.java`
within `pirate_phrases` and write the following:

```java
package pirate_phrases;

public class Greetings
{
    public static void hello()
    {
        System.out.println("Shiver me timbers!!!");
    }
}
```

Now create `Farewells.java` within the `pirate_phrases` directory:

```java
package pirate_phrases;

public class Farewells
{
    public static void goodbye()
    {
        System.out.println("A fair turn of the tide ter ye thar, ye bootlick slimebuckle!");
    }
}
```

If you navigate back to the parent directory of `pirate_phrases` and
run `javac PiratePhrases.java` followed by `java PiratePhrases`, you
should see this:

```java
Shiver me timbers!!!
A fair turn of the tide ter ye thar, ye bootlick slimebuckle!
```



# Notes

* You use local building materials (protocols, interfaces, file and
  date objs, compiling, exec jar file)
* Two perspectives:
    * the java objects perspective
        * Quick Java intro
        * instantiating
        * calling methods 
            * dot op
            * doto
        * class methods
        * navigating documentation
        * importing
        * commonly used classes
            * File
            * Date
            * Readers
            * Writers
            * System: getenv, exit, etc
    * the ops perspective
        * what it means to execute a java program (bytecode, jvm)
        * JVM as a platform (lots of library, analogous to GNU on
          linux)
        * compiling to bytecode
        * classpath
        * the ecosystem around java programs