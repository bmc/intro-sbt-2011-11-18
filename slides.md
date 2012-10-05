# An Introduction to SBT 0.10

<div markdown="1" id="logo">
<img src="ardentex-logo.png"/>
</div>

**Brian Clapper**\
**ArdenTex, Inc.**\
*@bclapper*\
*bmc@ardentex.com*

----------

# The sales pitch

* SBT is a _type-safe_ build environment.
* Define your build in terms of _expressions_, not inheritance hierarchies
  or XML.
* SBT 0.10 introduces a simpler `build.sbt` build file.
    + Simpler, but not necessarily simple

# Only `build.sbt` today

* In this talk, I'm going to focus on `build.sbt`
* ... largely because it's the newest thing ...
* ... and you'll be using it a lot.

----------

# Okay, I lied. A little.

* A _full configuration_ is more like an SBT 0.7 build file
* Can be mixed with a _quick configuration_ (a.k.a., `build.sbt`)
* File is `project/Build.scala`
* ... though anything ending in `*.scala` under `project` gets compiled
* You need `Build.scala` for multiproject projects.
* And that's all I'm going to say about it...

----------

# `build.sbt`

* Series of blank line-separated Scala clauses
* Each clause is independently evaluated
* In most cases, these clauses simply change settings

----------

# `build.sbt` Settings

* `Setting[T]` produces values of type `T`
* A build is a collection of `Setting` objects
* A setting can pull in other settings' result types
* Think of settings less as _running code_ and more as _configuring formulas_

----------

# `build.sbt` Settings

* Settings are modified with bind methods:
* := assignment
* += add one value
* ++= add sequence of values
* <~ transform
* <<= modify setting, using other settings' values
* <+= add value to setting, using other settings' values
* <++= add sequence value to setting, using other settings' values
* @#$%<sup>&amp;*(!</sup>
* More on these coming up.

----------

# New useful SBT shell commands

* `eval` - evaluate an expression
* `inspect` - inspect a task or setting and its dependencies
    + (Can be pretty damned unreadable to the uninitiated)
* `show` - run a task and show its output
* `last` - run the last command

----------

# Old useful SBT-isms, along for the ride

* `~` watch and repeat
* `+` run against cross version
* `++` run against all cross versions
* dependency DSL:
    + `"junit" % "junit" % "4.8" % "test"`

----------

# Aliases

    > alias q = exit
    > alias r = last
    > alias \!/ = compile

----------

# Settings, in more detail

## General form

    key in scope bind (dependencies) value

`bind` is a binding method.

### Example

    libraryDependencies <<= libraryDependencies apply {
      (deps: Seq[ModuleID]) =>
      // Note that :+ is a method on Seq that appends a single value
      deps :+ ("junit" % "junit" % "4.8" % "test")
    }

_Or just:_

    libraryDependencies <<= libraryDependencies apply { deps =>
      // Note that :+ is a method on Seq that appends a single value
      deps :+ ("junit" % "junit" % "4.8" % "test")
    }

----------

# Binding methods: :=

`:=` defines a setting that overwrites any previous value without referring
to other settings.

## Example:

Define a setting that will set `name` to "My Project", regardless of whether
`name` has already been initialized:

    name := "My Project"

No other settings are used. The value assigned is just a constant.

----------

# Binding methods: +=

`+=` defines a setting that appends a single value to the current sequence,
without referring to other settings.

## Example:

Define a setting that append a JUnit dependency to `libraryDependencies`.

    libraryDependencies += "junit" % "junit" % "4.8" % "test"

The dependency DSL should be familiar from sbt 0.7.

The `libraryDependencies` setting is used by many built-in tasks.

----------

# Binding methods: ++=

`++=` is similar to `+=`

Appends a _sequence_ to the current sequence, without using other settings

## Example:

Define a setting that will add dependencies on ScalaCheck and _specs_ to the
current list of dependencies:

    libraryDependencies ++= Seq(
      "org.scala-tools.testing" %% "scalacheck" % "1.9" % "test",
      "org.scala-tools.testing" %% "specs" % "1.6.8" % "test"
    )

----------

# some (unnecessary) geeky detail

Direct from the _sbt_ wiki:

> _The types involved in += and ++= are constrained by the existence of an
> implicit parameter of type `Append.Value[A,B]` in the case of `+=` or
> `Append.Values[A,B]` in the case of `++=`. Here, `B` is the type of the
> value being appended and `A` is the type of the setting that the value is
> being appended to. See `Append` for the provided instances._

Clear?

----------

# Binding methods: ~=

`~=` transforms the current value of a setting.

## Example:

Defines a setting that will remove `-Y` compiler options from the current
list of compiler options.

    scalacOptions in Compile ~= { (options: Seq[String]) =>
      options filterNot ( _ startsWith "-Y" )
    }

----------

# Binding methods: ~=

Thus, the earlier declaration of JUnit as a library dependency using `+=`
could also be written as:

    libraryDependencies ~= { (deps: Seq[ModuleID]) =>
      deps :+ ("junit" % "junit" % "4.8" % "test")
    }

----------

# Binding methods: <<=

`<<=` is the most general binding method.

Defines a setting using other settings, possibly including previous values
of the setting being defined.

All others can be implemented in terms of `<<=`

## Example:

Declare JUnit as a dependency using `<<=`:

    libraryDependencies <<= libraryDependencies apply { deps =>
      deps :+ ("junit" % "junit" % "4.8" % "test")
    }

----------

# Binding methods: <+=

`<+=` is a convenience hybrid of `+=` and `<<=`.

## Example:

Add a dependency on the Scala compiler to the current list of dependencies.

Because the `scalaVersion` setting is used, the method is `<+=`, not `+=`

    libraryDependencies <+= scalaVersion(
      "org.scala-lang" % "scala-compiler" % _
    )

----------

# Binding methods: <<=

`<++=` is a convenience hybrid of `++=` and `<<=`.

## Example:

Add a dependency on the Scala compiler to the current list of dependencies.

Because another setting (`scalaVersion`) is used and a `Seq` is appended,
use `<++=`

    libraryDependencies <++= scalaVersion { sv =>
      ("org.scala-lang" % "scala-compiler" % sv) ::
      ("org.scala-lang" % "scala-swing" % sv) ::
      Nil
    }

----------

# Tasks

* Tasks are like settings, except they don't cache.
* They're executed every time they're invoked.
* There's some talk of getting rid of settings entirely and making everything
  a task.

----------

# Using plugins

* The SBT Plugin API is completely new.
* Plugins have been migrating over. Slowly.
* The usage scenario is still in flux.
* General approach, though, defines keys within a scope. Same as regular SBT
  tasks and settings.

----------

# Using plugins

* To use a plugin, you refer to it as a dependency in 
  `project/plugins/build.sbt`
* SBT auto-imports the plugin object
* Examples of ported plugins:
    + sbt-assembly
    + sbt-izpack
    + sbt-lwm
    + sbt-editsource
    + sbt-growl-plugin
    + xsbt-proguard-plugin

* Many more here: <https://github.com/harrah/xsbt/wiki/sbt-0.10-plugins-list>

----------

# Let's see an example.

(cut to Emacs session...)

Example used is: <https://github.com/bmc/grizzled-scala/blob/master/build.sbt>

----------

# Credits and links

* Eugene Yokota, Doug Tangren, Rose Toomey: [Beginning SBT 0.10][]
* SBT GitHub page and Wiki: <https://github.com/harrah/xsbt/>
* _Full Configuration With SBT_: <http://www.fisharefriends.us/blog/2011-10-11-full-configuration-with-sbt/>


