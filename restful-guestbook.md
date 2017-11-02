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

### Linking to GitHub

Create a repository on GitHub. Note its URL (from the main repository
page), and then in the terminal in IntelliJ (opened via ALT-F12):

    $ git init
    $ git remote add origin https://github.com/<Remainder of URL>.git
    $ git pull origin master
    $ git push

and all the files in IntelliJ should now be in the remote repository.

### Looking Around

Taking a look at the structure that has now been created, we can
see that there is a single Java source file in `src/main/java/com/<yourname>/guestbook`. This is the file that
will kick off the complete Spring application. There is also a
folder called `resources` which contains a file
called `application.properties`, which as the name suggests is
where various options are set.

In the main folder is a file `pom.xml` which contains details of all
the dependencies for the project. For example, looking inside it there
should be an entry something like:

		<dependency>
			<groupId>com.h2database</groupId>
			<artifactId>h2</artifactId>
			<scope>runtime</scope>
		</dependency>

which is the dependency for an H2 database that was defined when the
project was created.

Maven will have been downloading these dependencies behind the scenes,
so it should now be possible to start the app off. It won't do
anything much as yet, but this will test that everything is set up
correctly.

The app can be started by the terminal command:

    $ mvn spring-boot:run

After a collection of messages, the app should be running on post
8080, so pointing a web browser at `localhost:8080` should reveal a
generic error message ("White Label") page.

##

