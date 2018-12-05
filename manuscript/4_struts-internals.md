# 4. Struts internals

## Core components

This chapter will allow you to understand how Struts works internally, what are the `bean`s, how everything is tied together. You will get knowledge how those components are co-operating together to handle request and execute your action, finally returning result to the browser.

## filters and prepare & execute phase

An entry point to your Struts application is defined via a servlet filter. You have two options:

- `org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter` - a commonly used filter which performs prepare and execute operations at the same time, this is a good choice for most of the applications
- `org.apache.struts2.dispatcher.filter.StrutsPrepareFilter` and `org.apache.struts2.dispatcher.filter.StrutsExecuteFilter` - these two filters will work exactly in the same way as the first filter but allows you to split prepare and operations into step which allows to use another filter in between to modify a servlet context. A good example is support for Sitemesh, which requires to modify the servlet context before an action can be executed.

As you can notice, there are two phases - preparation and execution. This is a default operation flow as from Struts 2.5. Such two steps process was introduced to allow alter the context by other filters before an action and its result will be executed. This gives you an opportunity to modify Struts logic using your own filter. 

Within the preparation phase the `Dispatcher` is initialized and properly configured based on current request (encoding and locale are set) and an `ActionContext` is created, which holds all the information needed to execute an action. And finally a proper `ActionMapping` is resolved.

The execution phase just executes the action that was already resolved in the prepare operation with the `ActionMapping`.

The are two classes that are responsible for each phase:

- `org.apache.struts2.dispatcher.PrepareOperations`
- `org.apache.struts2.dispatcher.ExecuteOperations`

You can use them as a staring point in developing your own filter with prepare and execution flow.

## action mapping
