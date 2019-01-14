# 2. Generate a project with Maven

## A word about Maven

Maven is a tool to easily manage your project. It is created under The Apache Software Foundation as the Apache Maven project and you can find more details visiting the webpage [https://maven.apache.org](https://maven.apache.org).
Maven can be used to manage dependencies (third party libraries) of your project, to build your project and create a WAR file or a HAR file. It also can help you run your project locally or deploy it to various environment. Also the main concept behind Maven is a _convention_ which means that different project use the same convention to store files, use the same folder hierarchy so this will allow you easily understand what is what and where to find a given file.

In this chapter I will just focus on creating a project using Maven, how to manage dependencies and how to run you application locally.

Before you will read the following sections and chapters please install Maven first. All the information how to download and install the toll you will find at [https://maven.apache.org](https://maven.apache.org).

## Struts Maven archetypes

A Maven archetype is a template that can be used to create a project. On the Internet you will find a plenty of the archetypes that you can use to generate your project. The simplest way to start generating a project is to use the below command:

```
mvn archetype:generate
```

After entering this command you will see that you can choose from over 1000 ready to be used archetypes - a think this is a way too much. Just cancel this process by pressing `CTRL+C`.

Let's start over. Here at Struts we have also prepared archetypes that can help you to start with your project. To narrow selection only to those archetypes please use the below command:

```
mvn archetype:generate -DarchetypeCatalog=https://struts.apache.org
```

As you can see there is one more option added at the end. It points Maven to our Archetype Catalog where only the Struts Archetypes can be found. After executing this command you will notice that you can have only 7 options to select from. Please select the first option named `Struts 2 Archetypes - Blank`.

Now you must enter a `groupId` - this can be a name of your company's domain or a home page, e.g. `pl.org.lenart` would be for me.
After entering the `groupId` and hitting enter you will have to define an `artifactId`. This is basically a name of your project or rather a code name. It supposed to be lowercase and with dashes, e.g. `struts-demo-1`. Hit enter to move to the next property. Now you must define a `version` of your project. We want be pay too much attention to it in this book so just hit enter accepting the the default value which is `1.0-SNAPSHOT`. And the very last property is a `package` which is by default equal to our `groupId` and yes this will be a name of out Java package in the project. You can accept the default choice or enter something more distinguishing like `pl.org.lenart.demo1`.

Now you will see all the properties entered during the process and you can accept them by hitting enter. After few seconds your project should be ready to use. The below message means everything went smoothly and you can start developing your project.

```
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

## Project structure

If you had never used Maven before I will explain folders and files structure now. See the structure below, this is a structure of the project we have created in the previous section:

```
.
├── pom.xml
└── src
    ├── main
    │   ├── java
    │   ├── resources
    │   └── webapp
    └── test
        └── java
```

This isn't the full structure, just three level deep but it shows the most important elements.

The very first element is a file `pom.xml`. This is a main configuration file for your project, to be clear, it isn't your Struts application configuration file. This file is the project configuration file where all the information about dependencies are stored, current version of you project and so on.

Below the `pom.xml` you can see a folder `src`. As you probably figured it out, it is a folder to store source files of your project. Then below we have to more folders, `main` and `test`. The `main` folder is used to store source code of your application logic where the `test` folder is used to store test files (unit tests, integration tests, etc).

On the lowest level of the `main` folder we have three folders:

- `java` - this folder is used to store your java source code e.g. `HelloWorld.java`
- `resources` - within this folder all the relates files are stored, like Struts configuration, properties files, and others, e.g. `struts.xml`, `package_pl.properties`
- `webapp` - and this folder is used to store files related to presentation layer like JSP files, CSS files, JavaScript files, etc. There is also a `WEB-INF` folder and `web.xml` file which you must be familiar with when building a Java web application. You can read [this](https://docs.oracle.com/javaee/6/tutorial/doc/bnadr.html) tutorial to understand more

Right now, if you would like to create a new Java code you must put your file in `src/main/java` folder, if this would be an unit test, it has to be created in `src/test/java` folder. If you are using a modern IDE like IntelliJ IDEA or Eclipse or NetBeans, those IDEs support Maven's folder structure out-of-the-box and will put a given type of file in a proper folder.

## Starting you app

As we have used a pre-configured archetype, our application is ready to be run and use. Maybe it doesn't have the functionality we expect but at least we have a starting point. To run your application locally type the bellow command inside the project folder:

```
mvn jetty:run
```

After few seconds you should see the below output which means your application is ready to be used:

```
...
[INFO] Started Jetty Server
[INFO] Starting scanner at interval of 10 seconds.
```

The archetype is using the Jetty Maven Plugin to start a Jetty container and run your application within it. For now you do not need any knowledge about the Jetty container, we won't be changing its default behavior.

Now you can open your web browser and point it to an address [http://localhost:8080/](http://localhost:8080/), you should see a web page like below:

```
Struts is up and running...

Languages
  English
  Espanol
```

and you should notice that the url has changed to `http://localhost:8080/example/HelloWorld.action`.

Success, your first Struts application is done!

To stop you application just hit `CTRL+C` in a terminal window.

## pom.xml

As I said previously `pom.xml` holds important information about your project. The most important are dependencies of your project which means what library are used in your project. You can easily control version of all the libraries in one place. Thus also allows upgraded them when needed. You can check for a new version of a library using [https://search.maven.org/](https://search.maven.org/).

See the below exercise:

```
<properties>
    <struts2.version>2.5.14.1</struts2.version>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
</properties>

<dependencies>

    <dependency>
        <groupId>org.apache.struts</groupId>
        <artifactId>struts2-core</artifactId>
        <version>${struts2.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.struts</groupId>
        <artifactId>struts2-config-browser-plugin</artifactId>
        <version>${struts2.version}</version>
    </dependency>

    <dependency>
        <groupId>org.apache.struts</groupId>
        <artifactId>struts2-junit-plugin</artifactId>
        <version>${struts2.version}</version>
        <scope>test</scope>
    </dependency>
    ...
```

The first section defines `properties` that can be used in your pom to keep the same information in one place. In this case you have information of currently used version of Struts extracted into a `<struts2.version>` property. By changing it, you will upgrade all the related Struts dependencies in one step. So let's do it, version `2.5.14.1` is a bit outdated so we must upgraded it to the latest available now which is `2.5.18`. Just replace `2.5.14.1` with `2.5.18` and you are done.

The following section defines project's dependencies. It's a list of `groupId/artifactId/version` - the same as you have been defining when creating your first project using the Struts Maven Archetypes. Those properties uniquely identify each dependency.

As you can notice there is one difference, the last dependency `struts2-junit-plugin` has `<scope>` property defined as `test`. It means that this dependency will be used only in tests (e.g. unit tests) and what's more important it won't be available in a production version of your application. Yes, this means there be no `struts2-junit-plugin.jar` in the final `WEB-INF/lib` folder. I will explain more about using test dependencies in chapter [9. Testing](#testing).

## Summary

In this chapter I wanted present some basic information about how to use Maven to manage your Java project. These days Maven is a basic tool for Java projects to manage dependencies and the whole life cycle of a Java project. You should be familiar with this tool as almost all the Apache Java projects are using it, also many other open source Java projects are using Maven.

Maven is very handy tool when used with Struts, we - Struts committers - are using it as well on daily basis when we are working on a new version of Struts. We consider this tool as must-have in terms of using Struts in your projects.
