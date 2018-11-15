# 3. Basic configuration

## What will be covered

In this chapter I will go through a `struts.xml` file and explain all the basic elements that can be used to configure your app. You can find this file in a `src/main/resources` folder of the example application which you have created in the previous chapter.

You can also use annotations or base the whole application on a convention, those are advanced topics and won't be covered in this chapter. There is also an option to configure the whole application programmatically. This is a new feature that was implemented in Struts 2.6. Please refer to the documentation for more information about those topics or wait for my another book.

## Structure

### include

Please open the `struts.xml` file in an editor or IDE of your choice. This file is entry point to configuration of your application, you can have more than one configuration file but all other files must be included in `struts.xml` to be loaded and parsed. You can to this by using `<include/>` tag as presented below:

```
<struts>

  <include file="struts-constants.xml"/>
  ...

  <include file="struts-actions.xml">

</struts>
 ```

You can use this directive to split configuration into smaller, logical blocks. You can have one file with all the constants, another with actions' configuration, and so on. Number and structure of the files depends on your application requirements. The largest the application the more blocks you must have - that's the rule of thumb. 

### constant

Another very important directive is `<constant/>` that is used to change behavior of the framework. The most known and important constant is `struts.devMode` which switches Struts into a development mode. In this mode, your application is more verbose, it will produce more logs also configuration will be automatically reloaded when changed, them same with properties files used to localize your application.

```
<struts>
  <constant name="struts.devMode" value="true">
</struts>
```

Since Apache Struts 2.5.6 you can use an env substitution mechanism which allows use environment variables or system properties in your constant definitions, e.g.:

```
<struts>
  <constant name="struts.devMode" value="${env.DEV_MODE:false}">
</struts>
```

As you see you can also specify a default value in case if the variable is missing, in this case `devMode` will be set to `false` when there is no `DEV_MODE` environment variable defined.

You can also define your own constants and inject them into your actions but this an advance topic that I will explore in my next book.

### bean

Struts allows defines beans, functional components, building blocks that are used to constitute the whole framework. Thus are also an extension points for the users. Each defined bean can be replaced with different implementation. That how plugins work, they provide a new version of given bean to change framework's behavior.

Here is an example bean definition from `struts-default.xml`:

```
<struts>
  <bean type="org.apache.struts2.dispatcher.multipart.MultiPartRequest" name="jakarta" class="org.apache.struts2.dispatcher.multipart.JakartaMultiPartRequest" scope="prototype"/>
</struts>  
```

The two most important attributes are:

  - `type` - an implementing interface
  - `class` - the exact implementation of the `type`

 This works very similar to other DI frameworks like Spring. The other attributes are:

  - `name` - an unique bean name, it cannot be duplicated across framework and in your application
  - `scope` - in which scope given bean should be created, `prototype` means that bean will be created on each invocation (not request) to get an instance of the bean. There is also scope `singleton` which means that there be just on instance of the bean across one instance of the framework.

  In most cases you will be selecting which bean to use be defining a constant. In this case to select other implementation of the `MultiPartRequest` you can define the constant as below:

```
<struts>
  <constant name="struts.multipart.parser" value="jakarta-stream"/>
</struts>
```

The `jakarta-stream` is a name of another implementation of a type `org.apache.struts2.dispatcher.multipart.MultiPartRequest` that is provided by Struts. You can find more in [Alternative Libraries](https://struts.apache.org/core-developers/file-upload.html#alternate-libraries) section in docs.

### package

Now we are moving into `package` which is a group of actions and interceptors that are tied to a given namespace. Each package is identified by `name` and `namespace` attributes. The `name` must be unique across your application. Namespace is just a logical name of your functionality, it isn't a URL or part of it. Having the following namespaces:

  - `/main`
  - `/main/admin`
  - `/main/user`

it doesn't mean that you have some hierarchy here, that `admin` and `user` namespaces are _beneath_ the `main` namespace. These are just logically separated blocks.

The other important attributes are:

  - `extends` - your package can extend an existing package to inherit all the defined actions, interceptors and so on. It is a good practice to define your own parent package (or even few packages) that you can use when defining the blocks. Using the one provided by the framework i.e. `struts-default` is a good idea only for small projects or when playing with Struts.
  - `abstract` - when set `true` it means that this package can be only used as a parent package of others, the same as with an abstract class in Java.
  - `strict-method-invocation` - this feature is set to `true` by default and it means that only defined action's methods will be exposed as actions. You can find more about that in my next book or in the [Security Guide](https://struts.apache.org/security/#strict-method-invocation)

