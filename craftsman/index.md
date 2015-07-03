---
title: Craftsman Introduction
---

Code Seed Craftsman generates concrete implementations for base classes whose abstract methods represent basic state. As an example: a base class named `Foo` with a single abstract method: `public abstract String getBar()` could have an `ImmutableFoo` object generated whose `getBar()` method always returns the same value.

Simple Example
--------------

By default, the configuration builder in Code Seed Configuration recognizes ordered annotations to define the "source" of configuration properties. For example:

Craftsman is annotation processor, primarily keyed off the `@Generate` annotation. For example:

{% highlight java %}
package com.example.myapp;
@Generate(Immutable.class)
public abstract class SomeObject {
    public abstract String someProperty();
}
{% endhighlight %}

The generated code might look something like:

{% highlight java %}
package com.example.myapp;
public final class ImmutableSomeObject extends SomeObject {
    private final String someProperty;
    
    ImmutableSomeObject(String someProperty) {
        this.someProperty = someProperty;	
    }
    
    @Override
    public String someProperty() {
        return someProperty;
    }
}
{% endhighlight %}

The generated code will be slightly more complex: for example, it will include `Object` methods like `hashCode`, `equals` and `toString`. In some cases there might be supplementary types (e.g. builders), in other cases the generated code will not directly extend the base type.

Generators
----------

The following code generators are included:

* _Immutable class_
  The generated class will be immutable (providing the base class does not maintain any state), it will have a builder and a factory method for copying state from another instance of the base class.
* _Bean class_
  The generated class will not extend the base class, instead it will be a proper bean with the ability to generate a dynamic view of the base type.
* _Forwarding class_
  The generated class will itself be abstract and will forward the abstract methods to another instance of the base type.

Because the code is actually generated, you can always see exactly what is being run.