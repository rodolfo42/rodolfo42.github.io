---
layout: post
title: Using Google Guice to manage environment profiles
date: 2012-09-09
comments: true
---

A few days ago, I started studying [MyBatis](http://www.mybatis.org/), a persistence framework for Java. Along its [XML configuration settings](http://www.mybatis.org/core/java-api.html#sqlSessions), there is a `<environment>` section available for specifying different environment profiles (development, test, production and the like). Basically, it looks like this:

{% highlight xml %}
<environments default="development">
    <environment id="development">
        <transactionManager type="JDBC">
        ...
        <dataSource type="POOLED">
        ...
    </environment>
    <environment id="production">
        <transactionManager type="MANAGED">
        ...
        <dataSource type="JNDI">
        ...
    </environment>
</environments>
{% endhighlight %}

As I started writing tests for a example webapp I was creating, I quickly felt the need to organize these different environment profiles using a simple mechanism, in order to easily get a `SqlSessionFactory` instance with the right environment setting whenever I needed. e.g: when testing, I needed an instance using the “test” environment profile.

<!--more-->

You can retrive a `SqlSessionFactory` instance by calling the `build()` method from the `SqlSessionFactoryBuilder` class, like so:

{% highlight java %}
String configFilename = "config/mybatis.cfg.xml";
InputStream inputStream = Resources.getResourceAsStream(configFilename);
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(inputStream);
{% endhighlight %}

which will construct a `SqlSessionFactory` that uses the default environment specified in the default attribute of the `<environments>` element. Optionally, you can specify the environment id as the second argument to the `build()` call:

{% highlight java %}
String environmentId = "test";
SqlSessionFactory factory = builder.build(inputStream, environmentId);
{% endhighlight %}

I first considered writing a very simple class with nothing but a static method like `getSqlSessionFactory(String env)`, that would return a `SqlSessionFactory` instance using the `environmentId` passed in by env.  But the downside is that each of my DAO classes, which all use `SqlSessionFactory` to open their sessions, would also have to carry this `environmentId` in order to get the right instance. Not to mention that:

1. they would all have to implement (or inherit) the logic of building a `SqlSessionFactory` using their `environmentId`
1. they would also need to be instantiated/configured by the caller code, and therefore the caller code would also have to know the `environmentId`

Instead of spreading the `environmentId` all over the place, I came up with a somewhat interesting solution using [Google Guice](http://code.google.com/p/google-guice/), which is a very handy way of using the [Dependency Injection](http://martinfowler.com/articles/injection.html) pattern. It’s lightweight and extremely easy to get it up and running.

First of all, I began applying the Dependency Injection pattern by turning my implicit dependency into an explicit one. This was easily done by requiring a ` SqlSessionFactory` instance to be provided via the constructor of all my DAO classes, like this:

{% highlight java %}
public abstract class AbstractDAO {
    protected SqlSessionFactory factory;
    protected SqlSession session;

    public AbstractDAO(SqlSessionFactory factory) {
        this.factory = factory;
    }

    protected void openSession() {
        session = factory.openSession();
    }

    // ..
}

public class SomeDAO extends AbstractDAO {

    public SomeDAO(SqlSessionFactory factory) {
        super(factory);
    }

    // ..
}
{% endhighlight %}

Again, I’ve only applied the dependency injection pattern so far. Note that my DAO classes now don’t need to know which `environmentId` or which XML file was used to instantiate the `SqlSessionFactory`, nor any other details of how it was instantiated. This kind of approach increases the cohesion and reusability of your code.

Next, I needed to configure Guice to inject an `SqlSessionFactory` instance into our DAO classes whenever they are constructed using a Guice `Injector`. The traditional approach to that **would** be:

{% highlight java %}
public class DAOModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(SqlSessionFactory.class).to(TestSqlSessionFactory.class);
    }
}
{% endhighlight %}

then, in the `setUp` method of your tests:

{% highlight java %}
Injector injector = Guice.createInjector(new DAOModule());
SomeDAO dao = injector.getInstance(SomeDAO.class);
{% endhighlight %}

where `TestSqlSessionFactory` would extend `SqlSessionFactory` and configure itself using the right `environmentId` string.

But that isn’t possible, since the `SqlSessionFactory` is instantiated using the `SqlSessionFactoryBuilder.build()` method, which require the config file input as the first argument to configure the instance that is returned.

Then I discovered something, just by analyzing the available methods that Eclipse suggested me right after `bind(SqlSessionFactory.class).` inside my `DAOModule`: a method called `toProvider()`, which maps your dependency to a class that implements `Provider<T>`, which provides an instance when needed. All you need implement is a method called `T get()` that returns an instance of your `T` dependency:

{% highlight java %}
public class TestSqlSessionFactoryProvider implements Provider<SqlSessionFactory> {
    public T get() {
        String configFilename = "config/mybatis.cfg.xml";
        String env = "test"; // environment id

        SqlSessionFactoryBuilder factoryBuilder = new SqlSessionFactoryBuilder();
        InputStream inputStream = new FileInputStream(configFilename);
        SqlSessionFactory factory = factoryBuilder.build(inputStream, env);
        return factory;
    }
}
{% endhighlight %}

Now each time I need a new setting for an environment, I just create a class that implements `Provider<SqlSessionFactory>` and configures/instantiates my `SqlSessionFactory` with the right environment setting. After some cleaning up, here’s what I came up with for my `Module`s:

{% highlight java %}
class DAOModule extends AbstractModule {
    public void configure() {
        bind(SqlSessionFactory.class)
            .toProvider(ProductionSqlSessionFactoryProvider.class);
    }
}

class TestDAOModule extends AbstractModule {
    public void configure() {
        bind(SqlSessionFactory.class)
            .toProvider(TestSqlSessionFactoryProvider.class);
    }
}
{% endhighlight %}

and on the `Provider` side:

{% highlight java %}
abstract class SqlSessionFactoryProvider implements Provider<SqlSessionFactory> {

    private String defaultConfigFilename = "config/mybatis.cfg.xml";

    public SqlSessionFactory get() {
        SqlSessionFactoryBuilder factoryBuilder = new SqlSessionFactoryBuilder();
        InputStream inputStream = new FileInputStream(getConfigFilename());
        SqlSessionFactory factory = factoryBuilder
            .build(inputStream, getEnvironmentId());
        return factory;
    }

    // override this to specify a different config filename
    public String getConfigFilename() {
        return defaultConfigFilename;
    }

    // should return something like: "test" or "production"
    abstract public String getEnvironmentId();
}

class ProductionSqlSessionFactoryProvider extends SqlSessionFactoryProvider {

    private String environmentId = "production";

    public String getEnvironmentId() {
        return environmentId;
    }
}

class TestSqlSessionFactoryProvider extends SqlSessionFactoryProvider {

    private String environmentId = "test";

    public String getEnvironmentId() {
        return environmentId;
    }
}
{% endhighlight %}

Now, inside my tests, I just need to tell Guice to load up the desired `Module`:

{% highlight java %}
class SomeDAOTest {
    private Injector injector = Guice.createInjector(new TestDAOModule());
    private SqlSessionFactory factory = null;

    @Before
    public void setUp() {
        factory = injector.createInstance(SomeDAO.class);
    }

    @After
    public void setUp() {
        factory = null;
    }

    // ..
}
{% endhighlight %}

It got a little verbose, we have long names like `ProductionSqlSessionFactoryProvider`, but doing that way eases maintenance, and prevents you from wasting time coming up with a unnecessarily short naming for your classes.