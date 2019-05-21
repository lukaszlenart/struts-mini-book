# 5. Validation

This chapter will help you understand how to use validation in your Struts application. You can define validation on your actions using a XML file or annotations. It is also possible to perform validation using your code in the actions.

## annotations

To perform a validation of action's field using annotations is simple, apply selected validation annotations to a related setter for the field.

```java
@RequiredFieldValidator(message = "You must enter a value for Age.")
@IntRangeFieldValidator(min = "18", max = "99", message = "Age must be between ${min} and ${max}, current value is ${age}.")
public void setAge(Integer age) {
    this.age = age;
}
```

In this case two annotations were used. First validates if value was defined at all, you must add the annotation if the value is required and not optional. Removing the `@RequiredFieldValidator` annotation means that the field is optional and doesn't have to have a value. If field was not defined in user request (a form field was left empty), the message attribute will be used to display an error message to the user.
You can also specify a `key` attribute that will be used to I18N the error message. As you can see you can use the validator's attributes in the message. Those attributes will be replaced with current values at runtime.

The `@IntRangeFieldValidator` expects that the int value will be in given range. In this case static values were used, but you also use expressions to calculate those values. Use `minExpression` and `maxExpression` attributes to define the expressions used to calculate the range for the value. This annotation also supports I18N, specify `key` attribute to show error message in user's language.

Each validation annotations has an attribute `shortCircuit`. When set to `true` the validation process will stop on the first unmet validation, which means user will see only one error message. In the example above, no range validation will be applied if `shortCircuit` would be set to `true`. You can use the attribute to stop validation at any point, it doesn't have to be the first annotation.

There is long list of available validation annotations, please read the [docs](https://struts.apache.org/core-developers/annotations.html#validation-annotations) to find them all.

## xml-based
