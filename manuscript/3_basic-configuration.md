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

