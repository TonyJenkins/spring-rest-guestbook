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

## Building the API

Our app will handle incoming requests (via HTTP) and will send back a
suitable (hopefully useful) response. We will use the `curl` program to
interact with the API from the command line. To check that `curl` is
installed, we can attempt to access the still-running app:

    $ curl localhost:8080

The response should be an error message (a 404 error code) in JSON
format, reporting that there is nothing found at the root page of
the app. It will look something like:

    {"timestamp":1509633964615,"status":404,"error":"Not Found","message":"No message available","path":"/"}⏎

### Adding a Route

A *route* can be thought of as some code that is executed when an
HTTP request is received at a particular URL.

The component of tha app that will manage routes is the
*Controller*. It is usual (but not compulsory) to store different
types of app component in different folders, so the first stage is
to create a folder called `controller` in the main source code
folder (`guestbook`) (IntelliJ calls this a "package". In that
folder, create a new class (right-click the folder, then New)
called `GuestBookController`. IntelliJ will fill in some skeleton code, and
not much more is needed to get a working controller.

First, annotate the class as a `RestController`:

     @RestController
     public class GuestBookController {
     .
     .
     }

Note that if all is working as it should, IntelliJ will automatically
find the correct `import` for the `@RestController` annotation.

Then we need to add a method that will handle a `GET` HTTP request
to the root of the app. For the moment it can just return a String:

    @GetMapping ("/")
    public String testMapping () {
        return "Hello, World";
    }

Restarting the app, and using `curl` to retrieve the root URL should
now result in this message being returned:

    $ curl localhost:8080
    Hello, World⏎

This basic route checks that everything is setup correctly, but before
it can do anything more interesting, a model is needed, along with
a database of some sort.

### Adding a Model

This app will use a simple in-memory database using H2. This could be
replaced by something like MySQL or MariaDB later on (and not very
much effort would be needed to make the change).

Before adding a model, we need to add a new dependency. Locate the POM
(`pom.xml` in the main top folder of the app), and edit it to include
the following along with all the current dependencies:

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-jpa</artifactId>
		</dependency>

Maven should now start downloading the new dependency.

To define the model, create a folder (package) called `domain`
alongside the `controller` folder. This package will contain two
classes: one will define the structure of the model
(that is, what is to be stored), and the second will define
methods that access the model (so, how the data once stored
is manipulated).

In `domain` create a class called `GuestBookEntry` to define the
data to be stored. Each instance of this class will correspond
to a stored record in the database. The class itself looks very
much like a standard Java class. There are two Strings, one for
the name of the user making the comment, and one for the comment
itelf, along with a unique identifier (an integer) to serve as
the primary key. The identifier will be maintained automatically,
and both Strings must be provided. So the attributes in the class
are defined as so:

    @Id
    @GeneratedValue (strategy = GenerationType.AUTO)
    private Integer id;

    @NotEmpty
    private String user;

    @NotEmpty
    private String comment;

The class itself, is annotated to show that it corresponds to an
entity in the database:

    @Entity
    public class GuestBookEntry {

The methods in the class can just be the usual constructors, getters,
setters, and a `toString`, all of which IntelliJ will generate
automatically.

The second class in the `domain` folder will define how
`GuestBookEntry` instances are retrieved from the database. There
is not much to include in this class (which will actually be an
interface) as Spring automatically provides most common query
requirements. The code below (imports omitted) defines
`GuestBookEntryRepository` as an interface that extends
`CrudRepository` and creates a method that will return all the
entries in the database as a standard Java `List`.

    public interface GuestBookEntryRepository extends CrudRepository <GuestBookEntry, Integer> {

        @Override
        List <GuestBookEntry> findAll ();

    }

The details of how the records are stored and retrieved will all be
handled by Spring.

Of course, the app cannot yet retrieve anything from the database
as there is nothing to connect the model and the controller. So we
need to finish things off by adding in the layer in between. To check
that the code to date compiles, the Maven command:

    $ mvn compile

will find all the source code, and compile.

### The Service Layer

The service layer can be thought of as sitting between the controller
and the domain. The controller will access methods from the service
layer, which in turn will call methods from the domain layer. The
reason behind this is *abstraction* in that the controller does not
need to know the details of the domain. Also, the service layer is
where we can implement any business logic, keeping this separate from
the domain, which can just concern itself with accessing the data
store.

To add the service layer, create a package called `service` alongside
the existing `domain` and `controller` packages. In this new
folder create a class called `GuestBookService` annotated with
`@Service` that has a `GuestBookRepository` as its attribue, and
that includes a single method that calls the `findAll`
method from `GuestBookRepository`.

    @Service
    public class GuestBookService {

        @Autowired
        private GuestBookEntryRepository guestBookEntryRepository;

        public List <GuestBookEntry> findAllEntries () {
            return this.guestBookEntryRepository.findAll ();
        }
    }

This new class is the first example of *dependency injection* we have
used. The `@Autowired` annotation declares that an instance of
GuestBookRepository` is required by the service layer; this instance
will be  created automatically when the app is started.

If everything compiles and looks OK, the final stage is to make
the controller call the service.


### Linking Controller and Service

There are two things to do to finsih off the first version of the app.
Firstly, the controller class now requires an instance of the service
class. This is dependency injection again, so we simply add an `@Autowired`
declaration to the controller class:

     @Autowired
     private GuestBookService guestBookService;

Then the current method can just be tweaked to call the method from the
service layer:

      @GetMapping ("/")
      public List <GuestBookEntry> testMapping () {
          return guestBookService.findAllEntries ();
      }

The app should now run, and `curl` can be used to access the route
defined. The result is an empty JSON string, because obviously there
is nothing yet in the database.

    $ mvn spring-boot:run
    $ curl localhost:8080
    []⏎

To prove that the app is working, the last piece is to configure
the database and then add some content. Happily, there is a quick
way to do this for testing purposes.

## Testing the App

Remember that the app currently uses an H2 in-memory database,
so the first stage is to have the database created when the app
starts. In the `resources` folder is a file called
`application.properties` where the database settings should be added.
The settings are:

    spring.datasource.url=jdbc:h2:mem:guestbook
    spring.datasource.username=sa
    spring.datasource.password=
    spring.datasource.driver-class-name=org.h2.Driver

Two further settings will allow the database to be investigated from
a web browser:

    spring.h2.console.enabled=true
    spring.h2.console.path=/h2

The H2 console is now available (when the app is running) at
`http://localhost:8080/h2`.

Of course, since the database is all in memory, anything added to the
Guestbook is lost when the app exits. This is fine for development
(and we could swap to a MySQL database later), but in order to see
if things are working we need to add some data. There is a quick and
easy way to do this, by creating an SQL script.

Still in the `resources` folder, create an SQL script called
`data.sql` in there that will create some sample data (the location
and the file name are important). For H2, the SQL is, for example:

    INSERT INTO GUEST_BOOK_ENTRY (COMMENT, USER) VALUES ('Great Comment', 'john');
    INSERT INTO GUEST_BOOK_ENTRY (COMMENT, USER) VALUES ('Me Too!', 'jane');
    INSERT INTO GUEST_BOOK_ENTRY (COMMENT, USER) VALUES ('I agree.', 'alice');

Here `GUEST_BOOK_ENTRY` is the table name that has been generated
automatically from the name of the class in the model. Likewise, the
database column names have been generated from the attribute names.
Both of these automatic generations can be overridden, but they will
do for now. In any case, this is the pretty much the only place where
the actual names of the table and columns will be needed.

Using `curl` to access the database should now result in a
promising JSON string containing the three Guestbook entries:

    $ curl localhost:8080
    [{"id":1,"user":"John Doe","comment":"Great Comment"},{"id":2,"user":"Jane Smith","comment":"Me Too!"},{"id":3,"user":"Alice Jones","comment":"I agree."}]⏎

We have an app!

## Tidying Up

There are a few things to do now that will tidy the code and make
things a little easier when adding more features to the app.

First, the table name in the database is not very developer-friendly.
Although we don't need it very often, we would probably prefer a name
in lower case, something like `entries`. This is easily specified by
changing the class definition for `GuestBookEntry` to specify the
table name we would prefer:

       @Entity
       @Table (name = "entries")
       public class GuestBookEntry {

Nothing else *in the code* needs to be changed, and as the table name
is currently hidden, the app will work exactly as before. The only
change needed is to alter the table name in the SQL `INSERT`
statements in the `data.sql` script.

A similar change can be used to give the `id` a more friendly name:

       @Id
       @GeneratedValue (strategy = GenerationType.AUTO)
       @Column (name = "entry_id")
       private Integer id;

Second, it is probably not the best idea to have the root of the
app return a complete list of all the entries. This simple fix changes
the route for a complete listing of the Guestbook. Change the only
route in the controller to:

       @GetMapping ("/comments")
       public List <GuestBookEntry> getAllComments () {
           return guestBookService.findAllEntries ();
       }

This code also changes the method name to something more useful,
although the actual name of the method is in this case quite irrelevant.

# Towards CRUD

A database app usually needs to implement four operations on its data
store:

* **C**reate new data.
* **R**rerieve existing data.
* **U**date existing data.
* **D**elete existing data.

We will look at each of these in the following sections. These
could be added in any order, but we will take them in (roughly)
increasing order of complexity: retrieval, deletion, creation and
updating.

## Retrieval

The app can already retrieve data from the database, so it makes
sense to start here. At present it can retrieve all the entries in the
guestbook; this will now be extended to two more precise queries:

* Using the entry's unique ID number to retrieve the corresponding
  entry (this will return exactly one entry, or possibly none at all
  if the ID is not found).
* Retrieving all comments by a certain user using the user name (this
  could return any number of entries, including none at all).

Both of these could be achieved by using the existing method to
retrieve all the records, and then writing some Java code to
process the resulting `List`. This would work, but it is a lot of
effort and breaks a golden rule - whenever possible, let the
database do the work.

### Retreiving an Entry bu ID

To retrieve a single entry by number, a URL such as
`http://localhost:8080/comment/1` would be good, where `1` is the
id of the entry in the database. The workflow to add this in is:

1. Create a query method in the Repository class that finds an
   entry by the ID.
2. Create a method in the service layer that calls this method in
   the Repository (and includes any relevant business logic).
3. Add a route to the Controller that calls the method in the
   service layer.

(This workflow is the same, in fact, for adding any new feature.)

So now, an HTTP GET request, sent via `curl`, will find the relevant
record.

For the first stage, add this method signature to the
`GuestBookEntryRepository` interface:

    GuestBookEntry findGuestBookEntryById (Integer id);

That's it. The method name itself is sufficient to express the
query required!

There is no business logic to add, so in the service layer
(class `GuestBookService`) the method just passes the results on:

And in the Controller, the URL is mapped with some extra code to
extract the required entry Id from the URL:

    @GetMapping ("/comment/{id}")
    public GuestBookEntry findGuestBookEntryById (@PathVariable ("id") Integer id) {
        return this.guestBookService.findGuestBookEntryById (id);
    }

The extra code here extracts `id` from the URL (as described in the
`@GetMapping`) and uses it as the parameter in the call to the
method.

So, now, the following should work, for example:

    $ curl localhost:8080/comment/1
    {"id":1,"user":"john","comment":"Great Comment"}⏎

