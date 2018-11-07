# 2. Generate a project with Maven

## A word about Maven

Maven is a tool to easily manage your project. It is created under The Apache Software Foundation as the Apache Maven project and you can find more details visiting the webpage [https://maven.apache.org](https://maven.apache.org).
Maven can be used to manage dependencies (third party libraries) of your project, to build your project and create a WAR file or a HAR file. It also can help you run your project locally or deploy it to various environment.

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

## 