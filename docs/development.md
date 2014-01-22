Development Environment
=======================

This document provides instructions for setting up a development environment. To set up the Vagrant development/testing VM, see https://github.com/stucco/dev-setup.

### Set up Eclipse/Git

1. Install Eclipse: Download [Eclipse IDE for Java EE Developers](eclipse.org/downloads/). Then open Eclipse and set up the default workspace somewhere on your drive.
2. Install e-git plugin: Help menu -> Eclipse Marketplace -> search for 'git' -> Install `EGit - Git Team Provider`

## RT (Storm) Setup

### Set up SBT (Simple Build Tool)

To be able to compile the project, run unit tests, and deploy jar files, you will need to download [SBT](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html). Be sure to get version >= 0.12.3.

### Get the project

The stucco-rt project is available here: [stucco-rt](https://github.com/stucco/rt)

Use `git clone` and `git pull` to get the most up-to-date versions of the projects.

Note that the project is a mixed source project. Any portion may be written in Java or Scala. Scala code can call Java code, and vice versa. Unit tests may be written in ScalaTest (which is preferred) or JUnit.

### Test the project

SBT makes it easy to compile, run, and deploy. Example usage is provided below. More information is available at the [SBT getting started guide](http://www.scala-sbt.org/release/docs/Getting-Started/Welcome.html).

    sbt compile     # compiles the project
    sbt test        # runs unit tests
    sbt assembly    # packages project into .jar file (for Storm CLI client)

## SBT

### SBT Plugins (Optional)

There are some SBT plugins that are quite useful. You can find more online, but here are some that we recommend you install:

* [Scalastyle](http://www.scalastyle.org/sbt.html)
* [Scalariform](https://github.com/sbt/sbt-scalariform)

### SBT Configuration

You can configure global SBT settings and plugins in your `~/.sbt` directory. The `~/.sbt/build.sbt` is included in all projects, and the `~/.sbt/plugins/plugins.sbt` are loaded for all projects. See the following examples for more details:

* [build.sbt](https://gist.github.com/anishathalye/6140974)
* [plugins.sbt](https://gist.github.com/anishathalye/6140962)

### SBT on a Mac

If `sbt` throws an error like `Error during sbt execution: java.lang.OutOfMemoryError: PermGen space`, you can create a `~/.sbtconfig` file with the following contents:

    SBT_OPTS="-XX:MaxPermSize=512M"%                                              

This is only tested on the `brew` version of sbt on Mac OS.