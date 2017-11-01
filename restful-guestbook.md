# A Spring RESFTful Guestbook

## Pre-requisites

Using this guide requires a Linux platform with the following toold
installed:

* Java 1.8 JDK
* *Git* for revision control.
* *Maven* for build automation and to manage dependencies.
* *Curl* for passing requests over HTTP.

JetBrains IntelliJ IDEA Ultimate Edition is also assumed.

GitHub (https://www.github.com/) is used for version control, but this
is optional, and any other Git service would do.

## Getting Started

There is quite bit to set up before getting started with Spring, and
this can be fiddly. Happily, a good toolset takes away much of the pain.
This guide uses IntelliJ to access Spring Initializr
(https://start.spring.io/) to create a template application.

Create a new project in IntelliJ, and pick "Spring Initializr" as the
project type. On the next screen pick `com.<yourname>` as the Group,
`guestbook` as the Artifact; leave the other fields or allow them to
change automatically. Finally, on the next screen, pick the Web
dependency (under `Web`), and the H2 database (under `SQL`). IntelliJ
will now send the date to the Spring Initializr, download the files it
generates, and open them up.

With this basic structure in place, it makes sense to connect the
IntelliJ project to a GitHub repository.

## Linking to GitHub

