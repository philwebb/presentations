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
        <wroFile>${basedir}/src/main/resources/wro/wro.xml</wroFile>
        <extraConfigFile>${basedir}/src/main/resources/wro/wro.properties</extraConfigFile>
    </configuration>
</plugin>
```

## Static Content - Wro4j with Maven

`src/main/resources/wro/wro.xml`

```xml
<groups xmlns="http://www.isdc.ro/wro">
  <group name="wro">
    <css>file:./src/main/resources/wro/main.less</css>
  </group>
</groups>
```

<p/>

`src/main/resources/wro/wro.properties`

```
postProcessors=less4j
```

<!---
We could jump to a wro4j demo here 
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



