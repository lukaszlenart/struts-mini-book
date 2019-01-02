# 4. Struts internals

## Core components

This chapter will allow you to understand how Struts works internally, what are the `bean`s, how everything is tied together. You will get knowledge how those components are co-operating together to handle request and execute your action, finally returning result to the browser.

## filters and prepare & execute phase

An entry point to your Struts application is defined via a servlet filter. You have two options:

- `org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter` - a commonly used filter which performs prepare and execute operations at the same time, this is a good choice for most of the applications
- `org.apache.struts2.dispatcher.filter.StrutsPrepareFilter` and `org.apache.struts2.dispatcher.filter.StrutsExecuteFilter` - these two filters will work exactly in the same way as the first filter but allows you to split prepare and operations into step which allows to use another filter in between to modify a servlet context. A good example is support for Sitemesh, which requires to modify the servlet context before an action can be executed.

As you can notice, there are two phases - preparation and execution. This is a default operation flow as from Struts 2.5. Such two steps process was introduced to allow alter the context by other filters before an action and its result will be executed. This gives you an opportunity to modify Struts logic using your own filter. 

Within the preparation phase the `Dispatcher` is initialized and properly configured based on current request (encoding and locale are set) and an `ActionContext` is created, which holds all the information needed to execute an action. And finally a proper `ActionMapping` is resolved.

The execution phase just executes the action that was already resolved in the prepare operation with an _action mapping_ component.

The are two classes that are responsible for each phase:

- `org.apache.struts2.dispatcher.PrepareOperations`
- `org.apache.struts2.dispatcher.ExecuteOperations`

You can use them as a staring point in developing your own filter if aim to modify the default prepare and execution flow.

## action mapping

Once request reached the filter, it's a time to map it to an action. An instance of `ActionMapper` is used the perform such operation. There are many different implementations of that interface, by default a `DefaultActionMapper` class is used. It use a standard `<action>.<extension>` pattern to extract an action from the request.

There are also two REST implementations in the Struts Core package - `Restful2ActionMapper` and `RestfulActionMapper`. You can find more about those two in the [documentation](https://struts.apache.org/core-developers/action-mapper.html).

There are also two meta mappers - `CompositeActionMapper` and `PrefixBasedActionMapper`. The first allows you to composite a few mappers and used the to map request to an action. It cycles throughout the configured list of mappers and uses the first non-null result. The second can be configure on per prefix basis, you specify a path prefix and a corresponding mapper.

To configure a different mapper than the default one, you must set the below constant to a proper value:

```xml
<constant name="struts.mapper.class" value="restful"/>
```

Where `name` must reference a bean implementing `org.apache.struts2.dispatcher.mapper.ActionMapper`.

## static content

If an ActionMapper could not identify an action in the request, it is also possible that the request points to a static resource that should be handled by Struts. Struts provides an interface `StaticContentLoader` which is used to handle such case.

There is also a default implementation - `DefaultStaticContentLoader` - which will handle requests starting either with `/struts/` or `/static/` prefix.

There are two important methods that must be implemented:

- `canHandle(String)` - indicates if this resource handler can handle the request and a potential static resource was identified in the request
- `findStaticResource(String, HttpServletRequest, HttpServletResponse )` - performs lookup to find the static resource, it can scan packages or JARs to fetch the static resource.

You can implement your own static resources handler, just define a bean implementing `org.apache.struts2.dispatcher.StaticContentLoader` interface and point struts to use it, as shown below:

```xml
<bean 
  type="org.apache.struts2.dispatcher.StaticContentLoader" 
  class="MyStaticContentLoader" 
  name="myLoader" />

<constant name="struts.staticContentLoader" value="myLoader" />
```

If your application should not serve static content, you can disable the whole functionality by setting a constant `struts.serve.static` to `false`:

```xml
<constant name="struts.serve.static" value="false" />
```

This option is useful if you serve static content out of Struts using a CDN mechanism or something like that. In such case is also a good idea to tell Struts to ignore requests that matches given pattern as they will be handled elsewhere:

```xml
<constant name="struts.action.excludePattern" value="/content/.*?" />
```

## object factory & internal DI

An object factory is responsible for creating any other objects defined by users. Thus means the object factory class will create new instances of actions, results, interceptors, validators and converters.

Just to remind you that actions are instantiated per request, so with each and every request a new instance will be created. There will be no shared state between the same two actions created from the same class. The same are results, which are created per request and do not share state.

Interceptors are singletons per package within they were defined and they should be able to handle shared state (if they use such functionality).

Converters and validators are normal singletons which are created once and reused every time when conversion or validation is needed. Thus requires to avoid using a shared state between them, those objects should be stateless.

The basic object factory implementation uses internal Dependency Injections (aka DI) mechanism to create instances of the classes. Just a site note, as from Struts 2.6 you can annotate an action's constructor and inject beans into it. Before it was not possible. There are other implementations that are using Spring or CDI to instantiate objects.

As I mentioned early, Struts provides its own small DI mechanism which is used internally to tie objects. It has no fancy mind-blowing functions like other DI frameworks. It's a pure essence of DI just to instantiate an object of a class and inject dependencies into it. Historically it is a predecessor of the [Guice Framework](https://github.com/google/guice).

The internal DI provides just one annotation: `@Inject` with two parameters: `value` which is a name of a dependency to inject and `required` which tells framework to ignore such dependency if missing.

A small example how to get access to an instance of the `ObjectFactory`:

```java
@Inject
public void setObjectFactory(ObjectFactory objectFactory) {
    this.objectFactory = objectFactory;
}
```

It looks very familiar as in Spring for example.

You can use internal DI to define your own beans and inject them into your actions. I wouldn't suggest using this technique in large applications as it can become cumbersome very fast and a lot of useful functions are missing. But it's a good idea for a small applications or just to play & learn how to use it.

You must start by defining a bean in `struts.xml`:

```xml
<struts>
  <bean 
    type="my.company.app.Interface" 
    class="my.company.app.MyImplementation"/>
</struts>  
```

It's exactly the same as I was presenting in chapter [Basic Configuration](#basic-configuration), but you can omit `name` and `scope` attributes. Then a default name will be used and scope will be defined as `singleton`. And now you can inject your bean into your action:

```java
public class MyAction {

  @Inject
  public MyAction(Interface impl) {
    this.impl = impl;
  }
  
  ...
}
```

As you can see, internal DI will select a proper object based on the defined type and will inject it when creating an action. To be honest, you can inject your bean in any other Struts class like interceptor, validator or converter.

## converters & validators
