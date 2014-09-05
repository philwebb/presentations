---
title: Spring Boot Intro
layout: springone14
---
# Spring Boot for the Web Tier

Phillip Webb  
twiiter: [@phillip_webb](http://twitter.com/phillip_webb)    
email: pwebb@pivotal.io   

Dave Syer   
twitter: [@david_syer](http://twitter.com/david_syer)   
email: dsyer@pivotal.io   


## Outline
* Static Content
* Dynamic Content
* Services and Messaging
* The Servlet Container
* Other Stacks

## Demo - Static Content

<!---
Demo here will show a simple index.html file being added
the mapping to '/' and an alternative favicon.ico. It will
also add show a webjar import. 
-->

## Static Content - Serving Files
* Can't use `/src/main/webapp` for jar deployments
* Put Static files from `src/main/resources/static`
* or `.../public` or `.../resources` or `.../META-INF/resources`

## Static Content - Conventions
* `src/main/resources/static/index.html` is mapped to `/index.html` & `/`
* Add a `src/main/resources/favicon.ico` to replace the Spring Leaf
* Imported "webjars" are automatically mapped

## Static Content - Processing Assets
* Use a JavaScript tool chain and create an assets jar
* Use a wro4j
  * JsHint
  * CssLint
  * JsMin
  * Google Closure compressor
  * UglifyJs
  * Less
  * Sass

## Static Content - Wro4j with Maven

```xml
<plugin>
    <groupId>ro.isdc.wro4j</groupId>
    <artifactId>wro4j-maven-plugin</artifactId>
    <version>${wro4j.version}</version>
    <executions><execution>
      <phase>generate-resources</phase>
      <goals><goal>run</goal></goals>
    </execution></executions>
    <configuration>
        <wroManagerFactory>ro.isdc.wro.maven.plugin.manager.factory.ConfigurableWroManagerFactory</wroManagerFactory>
        <destinationFolder>${basedir}/target/generated-resources/static/</destinationFolder>
        <wroFile>${basedir}/src/main/wro/wro.xml</wroFile>
        <extraConfigFile>${basedir}/src/main/wro/wro.properties</extraConfigFile>
    </configuration>
</plugin>
```

## Static Content - Wro4j with Maven

`src/main/wro/wro.xml`

```xml
<groups xmlns="http://www.isdc.ro/wro">
  <group name="wro">
    <css>file:./src/main/wro/main.less</css>
  </group>
</groups>
```

<p/>

`src/main/wro/wro.properties`

```
postProcessors=less4j
```

<!---
We could jump to a wro4j demo here 
-->

## Static Content: Grunt Toolchain

For serious front end developers the best choice is a Javascript
toolchain.

* Good community, lots of tools
* Package static assets into a jar
* And/or build them as part of a very thin back end
* Spring Boot CLI makes a great lightweight back end in production or for Java devs

<!--
Demo NPM toolchain. Show Spring Boot CLI app for backend.
-->

## Demo - Templating

<!--
Demo Groovy templates. Showing groovy code in-line.
-->

## Dynamic Content - Templating Support
* Thymeleaf
* Groovy Template Language
* Freemarker
* Velocity
* JSP (not recommended)

## Dynamic Content - Template Conventions
* Templates live in `src/main/resources/templates`
* and are accessed via `classpath:/templates/`
* Default Extensions are:
  * `*.html` - Thymeleaf
  * `*.tpl` - Groovy
  * `*.ftl` - Freemarker
  * `*.vm` - Velocity

## Dynamic Content - Template Customization
* User `spring.xxx.prefix` and `spring.xxx.suffix`
* eg. `spring.freemarker.suffix=fm`

## Dynamic Content - Custom Support
* Add a `ViewResolver`
* Add a `TemplateAvailabilityProvider`

```java
public class GroovyTemplateAvailabilityProvider implements TemplateAvailabilityProvider {

  @Override
  public boolean isTemplateAvailable(String view, Environment environment,
      ClassLoader classLoader, ResourceLoader resourceLoader) {
    if (ClassUtils.isPresent("groovy.text.TemplateEngine", classLoader)) {
      String prefix = environment.getProperty("spring.groovy.template.prefix",
          GroovyTemplateProperties.DEFAULT_PREFIX);
      String suffix = environment.getProperty("spring.groovy.template.suffix",
          GroovyTemplateProperties.DEFAULT_SUFFIX);
      return resourceLoader.getResource(prefix + view + suffix).exists();
    }
    return false;
  }

}
```

## Dynamic Content - Internationalization
* A `MessageSource` bean is added when `src/main/resources/messages.properties` exists
* Use `messages_LOCALE.properties` to add additional locales
  * e.g. `messages_FR.properties`
* Choose a specific locale using the `spring.mvc.locale` property
* Choose a specific date format using the `spring.mvc.date-format` property

<!-- We can demo adding messages to the template here -->

## Demo - Services

<!-- 
  Here we will demo a simple rest service then show adding a HtmlMessageConverter 
-->

## Services - Rest Controllers
* Use the `@RestContoller` annotation
* Use ResponseEntity builder methods with Spring Framework 4.1

```
ResponseEntity.
    accepted().
    contentLength(3).
    contentType(MediaType.TEXT_PLAIN).
    body("Yo!");
```

## Services - HttpMessageConverters
* Add `HttpMessageConverter` beans and Spring Boot will try to do the right thing
* It tries to be intelligent about the order
* Add a `HttpMessageConverters` bean if you need more control

## Other Stacks

* JAX-RS: Jersey 1.x, Jersey 2.x [dsyer/spring-boot-jersey](https://github.com/dsyer/spring-boot-jersey)
* Netty and NIO: Ratpack [dsyer/spring-boot-ratpack](https://github.com/dsyer/spring-boot-ratpack)
* Servlet 2.5 [scratches/spring-boot-legacy](https://github.com/scratches/spring-boot-legacy)
* Vaadin [peholmst/vaadin4spring](https://github.com/peholmst/vaadin4spring/tree/master/spring-boot-vaadin)

## Jersey 1.x

Easy to integrate with Spring Boot using `Filter` (or `Servlet`), e.g.

```
@Configuration
@EnableAutoConfiguration
@Path("/")
public class Application {

    @GET
    @Produces("text/plain")
    public String hello() {
        return "Hello World";
    }

    @Bean
    public FilterRegistrationBean jersey() {
        FilterRegistrationBean bean = new FilterRegistrationBean();
        bean.setFilter(new ServletContainer());
        bean.addInitParameter("com.sun.jersey.config.property.packages",
			"com.mycompany.myapp");
        return bean;
    }

}
```

(N.B. with fat jar you need to explicitly list the nested jars that have JAX-RS resources in them.)

## Jersey 2.x

Spring integration is provided out of the box, but a little bit tricky
to use with Spring Boot, so some autoconfiguration is useful. Example
app:

```
@Configuration
@Path("/")
public class Application extends ResourceConfig {

    @GET
    public String message() {
        return "Hello";
    }

    public Application() {
        register(Application.class);
    }

}
```

## Ratpack

> Originally inspired by Sinatra, but now pretty much
> diverged. Provides a nice programming model on top of Netty
> (potentially taking advantage of non-blocking IO).

2 approaches:

* Ratpack embeds Spring (and uses it as a `Registry`), supported natively in Ratpack 0.9.9
* Spring embeds Ratpack (and uses it as an HTTP listener) = spring-boot-ratpack

## Spring Boot embedding Ratpack

Trivial example (single `Handler`):

```
@Bean
public Handler handler() {
    return (context) -> {
        context.render("Hello World");
    };
}
```

## Spring Boot embedding Ratpack

More interesting example (`Action<Chain>` registers `Handlers`):

```
@Bean
public Handler hello() {
    return (context) -> {
        context.render("Hello World");
    };
}

@Bean
public Action<Chain> handlers() {
    return (chain) -> {
        chain.get(hello());
    };
}
```

## Spring Boot Ratpack DSL

A valid Ratpack Groovy application:

```
ratpack {
  handlers {
    get {
      render "Hello World"
    }
  }
}
```

launched with Spring Boot:

```
$ spring run app.groovy
```
