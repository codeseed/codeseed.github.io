---
title: Configuration Introduction
---

Code Seed Configuration helps you separate the collection and transformation of configuration information from the rest of your code. The idea is simple: write an interface describing the configuration you need and write your code to that; then use Code Seed Configuration to provide an implementation of that interface once you are ready to start deploying your application.

Simple Example
--------------

By default, the configuration builder in Code Seed Configuration recognizes ordered annotations to define the "source" of configuration properties. For example:

{% highlight java %}
package com.example.myapp;
public interface MyAppConfig extends Configuration {
    @SystemProperty
    @EnvironmentVariable
    String serverHost();
}
{% endhighlight %}

You can expect an implementation of this interface to work something like:

{% highlight java %}
package com.example.myapp;
public class MyAppConfigImpl implements MyAppConfig {
    @Override
    public String serverHost() {
        String systemProperty = System.getProperty("com.example.myapp.serverHost");
        if (systemProperty != null) {
            return systemProperty;
        }
        
        String environmentVariable = System.getenv("SERVER_HOST");
        if (environmentVariable != null) {
            return environmentVariable;
        }
        
        // Suspend disbelief, pretend getMethod cannot fail
        throw new MissingPropertyException(MyAppConfig.class.getMethod("serverHost"));
    }
}
{% endhighlight %}

But who wants to write all that code?

{% highlight java %}
MyAppConfig config = ConfigurationBuilder.newBuilder().build(MyAppConfig.class);
{% endhighlight %}

There are also `@SystemPreference` and `@UserPreference` annotations; for times when you need a fallback, there is `@DefaultValue`. Optional third party integrations allow you to source configuration properties from other sources: for example, [Apache Commons CLI][apache-commons-cli] enables the use of the `@CommandLine` annotation. Finally, you can always implement the `PropertySource` interface with any code you want.

Slightly More Complex Example
-----------------------------

Let's make the configuration a little more interesting:

{% highlight java %}
package com.example.myapp;
import java.util.Locale;
import java.nio.file.Path;
import org.codeseed.common.config.*;
public interface MyAppConfig extends Configuration {
    @SystemProperty
    @EnvironmentVariable
    @DefaultValue("localhost")
    String serverHost();
    
    @SystemProperty
    @EnvironmentVariable
    @DefaultValue("8080")
    int serverPort();
    
    @EnvironmentVariable("MY_APP_HOME")
    @DefaultValue("${user.dir}")
    Path home();
    
    @CommandLine({ "v", "verbose" })
    boolean verbose();
    
    @UserPreference
    Locale lang();
}
{% endhighlight %}

To get `@CommandLine` to work we need Apache Commons CLI and a little bit of extra code:

{% highlight java %}
package com.example.myapp;
import java.util.ListResourceBundle;
import org.codeseed.common.config.*;
import org.codeseed.common.config.ext.*;
public class MyApp {
    public static class Bundle extends ListResourceBundle {
        protected Object[][] getContents() {
            return new Object[][] = {
                {"verbose.description", "be verbose"},
            };
        }
    }

    public static void main(String[] args) {
        MyAppConfig config = ConfigurationBuilder.newBuilder()
            .with(CommandLineHelper.posix("com.example.myapp.MyApp$Bundle")
                 .withHelpOption()
                 .withOptions(MyAppConfig.class)
                 .parse(args))
            .withDefaultValues()
            .build(MyAppConfig.class);
    }
}
{% endhighlight %}

The addition of the `withDefaultValues()` also ensures that primitives and value types with system defaults (like `Locale`) get their default values.

[apache-commons-cli]: http://commons.apache.org/proper/commons-cli/