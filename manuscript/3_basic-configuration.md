# 3. Basic configuration

## What will be covered

In this chapter I will go through a `struts.xml` file and explain all the basic elements that can be used to configure your app. You can find this file in a `src/main/resources` folder of the example application which you have created in the previous chapter.

You can also use annotations or base the whole application on a convention, those are advanced topics and won't be covered in this chapter. There is also an option to configure the whole application programmatically. This is a new feature that was implemented in Struts 2.6. Please refer to the documentation for more information about those topics or wait for my another book.

## Structure

### include

Please open the `struts.xml` file in an editor or IDE of your choice. This file is entry point to configuration of your application, you can have more than one configuration file but all other files must be included in `struts.xml` to be loaded and parsed. You can to this by using `<include/>` tag as presented below:

```xml
<struts>

  <include file="struts-constants.xml"/>
  ...

  <include file="struts-actions.xml">

</struts>
 ```

You can use this directive to split configuration into smaller, logical blocks. You can have one file with all the constants, another with actions' configuration, and so on. Number and structure of the files depends on your application requirements. The largest the application the more blocks you must have - that's the rule of thumb. 

### constant

Another very important directive is `<constant/>` that is used to change behavior of the framework. The most known and important constant is `struts.devMode` which switches Struts into a development mode. In this mode, your application is more verbose, it will produce more logs also configuration will be automatically reloaded when changed, them same with properties files used to localize your application.

```xml
<struts>
  <constant name="struts.devMode" value="true">
</struts>
```

Since Apache Struts 2.5.6 you can use an env substitution mechanism which allows use environment variables or system properties in your constant definitions, e.g.:

```xml
<struts>
  <constant name="struts.devMode" value="${env.DEV_MODE:false}">
</struts>
```

As you see you can also specify a default value in case if the variable is missing. In this case `devMode` will be set to `false` when there is no `DEV_MODE` environment variable defined.

You can also define your own constants and inject them into your actions but this an advance topic that I will explore in my next book.

### bean

Struts allows to define beans, functional components, building blocks that are used to constitute the whole framework. Thus are also an extension points for the users. Each defined bean can be replaced with different implementation. That how plugins work, they provide a new version of given bean to change framework's behavior.

Here is an example bean definition from `struts-default.xml`:

```xml
<struts>
  <bean 
    type="org.apache.struts2.dispatcher.multipart.MultiPartRequest" 
    class="org.apache.struts2.dispatcher.multipart.JakartaMultiPartRequest"
    name="jakarta"
    scope="prototype"/>
</struts>  
```

The two most important attributes are:

  - `type` - an implementing interface
  - `class` - the exact implementation of the `type`

 This works very similar to other DI frameworks like Spring. The other attributes are:

  - `name` - an unique bean name, it cannot be duplicated across framework and in your application
  - `scope` - in which scope given bean should be created, `prototype` means that bean will be created on each invocation (not request) to get an instance of the bean. There is also scope `singleton` which means that there be just on instance of the bean across one instance of the framework.

  In most cases you will be selecting which bean to use be defining a constant. In this case to select other implementation of the `MultiPartRequest` you can define the constant as below:

```xml
<struts>
  <constant name="struts.multipart.parser" value="jakarta-stream"/>
</struts>
```

The `jakarta-stream` is a name of another implementation of a type `org.apache.struts2.dispatcher.multipart.MultiPartRequest` that is provided by Struts. You can find more in [Alternative Libraries](https://struts.apache.org/core-developers/file-upload.html#alternate-libraries) section in docs.

### package

Now we are moving into `package` which is a group of actions and interceptors that are tied to a given namespace. Each package is identified by `name` and `namespace` attributes. The `name` must be unique across your application. Namespace is just a logical name of your functionality, it isn't a URL or a part of it. Having the following namespaces:

  - `/main`
  - `/main/admin`
  - `/main/user`

it doesn't mean that you have some hierarchy here, that the `admin` and `user` namespaces are _beneath_ the `main` namespace. These are just logically separated blocks.

The other important attributes are:

  - `extends` - your package can extend an existing package to inherit all the defined actions, interceptors and so on. It is a good practice to define your own parent package (or even few packages) that you can use when defining the blocks. Using the one provided by the framework i.e. `struts-default` is a good idea only for small projects or when playing with Struts.
  - `abstract` - when set `true` it means that this package can only be used as a parent package of others, the same as with an abstract class in Java.
  - `strict-method-invocation` - this feature is set to `true` by default and it means that only defined action's methods will be exposed as actions. You can find more about that in my next book or in the [Security Guide](https://struts.apache.org/security/#strict-method-invocation)

### result-types and result-type

Results are defined types of responses your application can perform as a final outcome of your action. Struts provides few pre-defined results:

  - `dispatcher` - a forward to a view, in most cases a JSP file
  - `tiles` - process a result as a Tiles definition
  - `httpheader` - returns response as HTTP headers
  - `stream` - response is a stream that can be downloaded by a browser

and many more. Also plugins can provide new types of results. You can find more information in [Result Types](https://struts.apache.org/core-developers/result-types.html) section of the documentation.

As I mentioned early, your application should define a parent package that will be reused across the application. In this package you should define the results that will be available to your actions. It's a rare case that an application will use all the available results, in most cases applications are using just few of them. And those results should be strictly defined in the parent package.

Below is an example of pre-defined set of results for your application:

```xml
<result-types>
  <result-type 
    name="dispatcher" 
    class="org.apache.struts2.result.ServletDispatcherResult" 
    default="true"/>
  
  <result-type 
    name="httpheader" 
    class="org.apache.struts2.result.HttpHeaderResult"/>
</result-type>  
```

As you see a result definition consist of a `name` attribute which can be an arbitrary string, it doesn't have to be the same as in the Struts. You can use a name that fits your requirements best. The second required attribute is `class` which must point to an existing Java class that implements `Result` interface.

The last attribute is `default` which tells framework which result to use when there was no result type definition in the action definition. I will explain this below in a section about actions.

### interceptors and interceptor

Interceptors are core concept of Struts to control flow of a request. They are used to control how incoming request should be steered to a given action and then to the final result. Below you can see how interceptors can affect action flow:

![](images/interceptors.png)

As you see interceptors are executed before an action and after it, so they can change the flow. This is a very useful functionality that can be used e.g. to intercept unauthorized request and redirect them to a login page. Or you can use an interceptor to record all the request parameters into a log. You can find more in [Interceptors](https://struts.apache.org/core-developers/interceptors.html) section of the documentation.

The same as for results, in most cases your application doesn't require to use all af the available interceptors. Thus can affect performance of your application if there is too many interceptors that must process the request. Also interceptors can have cross-dependencies between each other. That's why you should just define a dedicated set of interceptors that will be used in your application. This will also allow defining your custom interceptors in the future.

Below is an example of custom set of interceptors:

```xml
<interceptors>
  <interceptor name="exception" 
    class="com.opensymphony.xwork2.interceptor.ExceptionMappingInterceptor"/>
  <interceptor name="servletConfig" 
    class="org.apache.struts2.interceptor.ServletConfigInterceptor"/>
  <interceptor name="i18n" 
    class="org.apache.struts2.interceptor.I18nInterceptor"/>
  <interceptor name="params" 
    class="com.opensymphony.xwork2.interceptor.ParametersInterceptor"/>
  <interceptor name="prepare" 
    class="com.opensymphony.xwork2.interceptor.PrepareInterceptor"/>
  <interceptor name="conversionError" 
    class="org.apache.struts2.interceptor.StrutsConversionErrorInterceptor"/>
  <interceptor name="validation" 
    class="org.apache.struts2.interceptor.validation.AnnotationValidationInterceptor"/>
  <interceptor name="workflow" 
    class="com.opensymphony.xwork2.interceptor.DefaultWorkflowInterceptor"/>
  <interceptor name="fileUpload" 
    class="org.apache.struts2.interceptor.FileUploadInterceptor"/>

  <interceptor name="login" 
    class="com.gruuf.web.interceptors.LoginInterceptor"/>
  <interceptor name="user" 
    class="com.gruuf.web.interceptors.UserInterceptor"/>
  <interceptor name="auth" 
    class="com.gruuf.web.interceptors.AuthInterceptor"/>
  <interceptor name="secure" 
    class="com.gruuf.web.interceptors.SecureInterceptor"/>
  ...
```

This excerpt comes from my own application and as you see I'm just using few interceptors provided by the framework, beneath are my custom interceptors. Similar to the result definition, an interceptor definition consist of a `name` attribute and a `class` attribute. You can use an arbitrary name and the class must point to a Java class which implements `Interceptor` interface.

### interceptor-stack and interceptor-ref

All the interceptors are collected into stacks and in your Struts application you will be using those stacks to adjust behavior of one given part of the application to expected needs. Yes, you are right, you should have few custom stacks that fits your requirements. You shouldn't just use the existing stacks provided by the framework. Use them as an example to define your own stacks. It's the same case as with results, it rarely happens that one interceptors stack will fit into all the circumstances.

Below is an example stack build using the above defined interceptors:

```xml
<interceptor-stack name="gruufDefault">
  <interceptor-ref name="exception">
    <param name="logEnabled">true</param>
    <param name="logCategory">com.gruuf</param>
    <param name="logLevel">error</param>
  </interceptor-ref>
  <interceptor-ref name="secure"/>
  <interceptor-ref name="about"/>
  <interceptor-ref name="login"/>
  <interceptor-ref name="auth"/>
  <interceptor-ref name="servletConfig"/>
  <interceptor-ref name="user"/>
  <interceptor-ref name="restriction"/>
  <interceptor-ref name="policy"/>
  <interceptor-ref name="i18n"/>
  <interceptor-ref name="params"/>
  <interceptor-ref name="prepare"/>
  <interceptor-ref name="params"/>
  <interceptor-ref name="conversionError"/>
  <interceptor-ref name="validation">
    <param name="excludeMethods">input,back,cancel,browse</param>
  </interceptor-ref>
  <interceptor-ref name="workflow">
    <param name="excludeMethods">input,back,cancel,browse</param>
  </interceptor-ref>
</interceptor-stack>
```      

As you can see, to define a stack you must use `<interceptor-stack/>` tag and define a `name` which must be unique across the application. You build your stack using `<interceptor-ref/>` tag with a `name` attribute pointing to previously defined interceptor. You can create stacks from existing stack to slightly modify its behavior by adding another interceptor, see the example below:

```xml
<interceptor-stack name="defaultWithMessages">
  <interceptor-ref name="gruufDefault"/>
  <interceptor-ref name="store">
    <param name="operationMode">AUTOMATIC</param>
  </interceptor-ref>
</interceptor-stack>
```

I have used the previously defined stack `gruufDefault` and created a new stack by adding one more interceptor to the end. It's a good practice to create your own stacks. Please remember that sometimes there inner dependencies between interceptors, which means that one interceptor must be put next after another to work properly.

### default-*

There are few defaults that you should define to allow your application work smoothly.

The first is `<default-interceptor-ref/>` tag, which defines which interceptor stack to use if no one was defined for an action. This is a per package configuration so you can have different defaults for different packages.

```xml
<default-interceptor-ref name="gruufDefault"/>
```

You must just reference an existing stack that was defined in this package or in any other parent package. This means if you have an action `hello` without interceptors definition, the default interceptor reference will be used. I will show an example later when I will be presenting an action's configuration.

Another default-* is `<default-action-ref/>` which tells framework which action to execute if none match the request in the package. You can think of this default action as an `index.html` file which is by default served by a web server. The default action reference has the same purpose, framework will execute the default action if request matched package's namespace but no action.

```xml
<default-action-ref name="index"/>
```

With a `name` attribute you must reference an existing action definition in this package or in the parent packages.

The last default-* is `<default-class-ref/>` which defines a Java class that will be used to instantiate actions without a `class` attribute. I will explain this more when I will be showing you how to define an action.

```xml
<default-class-ref class="com.gruuf.web.actions.BaseAction"/>
```

The `class` attribute must point to an existing Java class and class must be an instantiable Java class (neither an interface nor an abstract class).

### global-results

Defining a `<global-results/>` is the same as defining a normal `<result/>` only scope is wider. In this case it's a package scope, which means you can use the defined global results with actions defined in this package or in any other package that inherits of this package.

```xml
<global-results>
  <result name="https-redirect" type="redirect">
    <param name="location">https://gruuf.com/</param>
  </result>
  <result name="login" type="redirectAction">
    <param name="actionName">login</param>
    <param name="namespace">/</param>
  </result>
  <result name="error">error</result>
  ...
```

The `<global-results/>` is a list of defined `<result/>`s, that's all. I will explain how to define a `<result/>` wider in a section below.

### global-allowed-methods

This is a new functionality introduced with Struts 2.5, it allows define methods that can be accessible with a bang, e.g. `Category!create.action`. Using `!` is a technic called [Dynamic Method Invocation](https://struts.apache.org/core-developers/action-configuration.html#dynamic-method-invocation) and limiting access to which method can be used with the `!` is called [Strict Method Invocation](https://struts.apache.org/core-developers/action-configuration.html#strict-method-invocation). This is an advanced security topic and you shouldn't change this if you are not 100% sure of what you doing. Defining a wrong `<global-allowed-methods/>` with DMI enabled can lead to a security vulnerability of your application.

Here is a list of methods defined in `struts-default.xml`

```xml
<global-allowed-methods>
  execute,
  input,
  back,
  cancel,
  browse,
  save,
  delete,
  list,
  index
</global-allowed-methods>
```

In most cases you don't have to define those methods, the default is enough.

### global-exception-mappings and exception-mapping

Defining a `<global-exception-mappings/>` is a good practice to properly handle exceptional situations in your application. This allows show a proper response in case your application has crashed. As name suggest those are global handlers that will be used if there were no `<exception-mapping/>` defined for an action.

```xml
<global-exception-mappings>
  <exception-mapping 
    exception="java.lang.Exception" result="error" />
</global-exception-mappings>
```
To define a handler for a given type of exception you must provide a full name of the Java exception class in the `exception` attribute. In the example above a general `java.lang.Exception` class is used which means this will catch any exception excepts errors. The `result` attributes points to a global result defined within this package or in any parent package.

Basically, when an exception occurs the framework will stop processing current request and will steer flow to the defined result. The exception will be available on the stack and can be used to show more details about it:

```xml
<h4>Exception Name: <s:property value="exception" /> </h4>
<h4>Exception Details: <s:property value="exceptionStack" /></h4>
```

### action

Finally we are moving to a core Struts functionality. Actions are the central points of your application when developing a Struts based application. It's a layer where your web logic happens. You shouldn't mix your web logic with your business logic, the former should be delegated to beans, services or DAOs - depending on how your application is structured.

Yet, you can put all your logic in actions only if you are testing a solution or just playing with Struts. You should always have clear boundaries between layers performing different logic.

Let's define a simple action.

```xml
<action name="index"/>
```

To define an action you must use a `<action/>` tag with at least one attribute `name` - the `name` must be unique in the given namespace. You don't have to define `<result>`s if your action will use one of the global results defined previously.

Let's analyze this super simple action config. What class will be used to instantiate the action? Do you remember `<default-class-ref/>` for the previous sections? If so, exactly that class will be used to create instance of the `index` action. By default Struts defines `<default-class-ref>` as:

```xml
<default-class-ref class="com.opensymphony.xwork2.ActionSupport" />
```

If didn't define your own default, this class will be used but only if your package extends `struts-default` package.

Another questions, what stack of interceptors will be applied to this action? Do you remember `<default-interceptor-ref/>`? Right, exactly this stack of interceptors will be applied to the action. And again, if you are extending `struts-default` package, `defaultStack` is defined as the default interceptors stack.

Another question, what method will be called inside the `ActionSupport` class? This isn't something that you can define but Struts has built-in default to execute an `execute` method when none was provided. This is also the first method of the `<global-allowed-methods>` defined in `struts-default.xml` as showed above.

As you see, Struts will do a lot more than you expect based on such a simple configuration. That's why knowledge about different aspects of Struts configuration is so important to properly understand how to configure actions.

Let's configure one more Struts action, this time a bit more complicated:

```xml
<action name="edit" class="org.apache.struts2.showcase.action.SkillAction" method="edit">
    <result name="input">/WEB-INF/empmanager/editSkill.jsp</result>
    <result type="redirect">list.action</result>
    <interceptor-ref name="params" />
    <interceptor-ref name="basicStack"/>
</action>
```

Here we have an action named `edit` which is an instance of the class `SkillAction` and a method `edit` will be called to execute the action. So `SkillAction` must have the method `edit` that will return a `String` value that will be mapped onto defined action results and if not found, onto the global results.

Please note that you do not have to define `edit` as an allowed action method (see `global-allowed-methods` section above). Defining a `method` attribute makes the method an allowed method automatically, that how SMI works.

The action defines two action results - `input` and a default one (no name). The default one is `success` because Struts assumes `success` as a default result name. So your action can either return `success` or `input`. The `input` result doesn't specify `type` which means a default type will be used which is `dispatcher` - the default is defined by setting a `default` attribute to `true` when defining a result (see section above about `result-types`).

Next, there are two references to interceptors defined, where the first is an interceptor named `params` and the second is an interceptors stack called `basicStack` (which is defined in `struts-default.xml`)

Basically what we have did here is to define some non-default values that will be used instead of the framework predefined defaults. Any other action configurations do the same - replaces default with action's specific values.

### unknown-handler-stack & unknown-handler-ref

The final configuration options are `<unknown-handler-stack/>` and `<unknown-handler-ref/>` which can be used to handle an unknown action. An action is unknown if it didn't match any defined action in a given package. The handlers will be called just before steering flow into `<default-action-ref/>`. These handlers can be also used to handle an unknown result and action method.
The same as interceptors you can stack many handlers into stacks, but only one stack can be defined per package.

A `<unknown-handler-ref/>`'s `name` attribute is a reference to a `bean`, which is an implementation of the `UnknownHandler` interface. You can inspect the interface to see its methods and basically returning a `null` means the action, result or method is still unknown.

If there is no stack defined in the package but unknown handlers were defined, they will be loaded and applied in a random order.

## Summary

As you can see, configuring Struts is very flexible. You can change almost any aspect of the framework by implementing a specific interface and defining it as a new `bean`. Also in most cases you are changing the existing default behavior by simply adjusting few config options. As you were able to see how to configure actions, you can now understand that action's configuration heavily depends on tweaking the defaults. Understanding this will help you better understand your project and how to use Struts to achieve expected results in the simplest manner.
