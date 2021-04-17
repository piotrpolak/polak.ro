---
layout: single
author: Piotr Polak
title: "API First and code generation done right"
read_time: true
toc_sticky: true
toc: true
date: 2021-04-02 17:52:56 +0200
last_modified_at: 2021-04-17 19:38:00 +0200
author_profile: true
---

This document is written from a Java/Fullstack developer perspective and it is intended to explain API First approach
from the business, consumer, and API creator points of view.

## Some historical background

In past days HTTP APIs used to be either random collections of RPC-like endpoints of some heavy SOAP interfaces.

Some time around 2000 Roy Fielding presented REST in his doctoral dissertation but it took some time
before it was adopted as a common practice.

For the enterprise world SOAP was a default choice as an integration protocol as it had a great advantage over REST
- SOAP web services were formally defined by **machine-readable description language** - WSDL
- which allowed the code generation tools to take care of all the **glue code**.

SOAP was not widely adopted by web and mobile applications due to its implementation complexity and high verbosity.

{% capture fig_img %}
![The Internet](/assets/2021-04-02-api-first-done-right/the-internet.jpg)
{% endcapture %}
{% capture fig_caption %}
This is the Internet
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>{{ fig_caption | markdownify | remove: "<p>" | remove: "</p>" }}</figcaption>
</figure>

### The adaptation of Swagger and the SwaggerUI

In 2011 Tony Tam created an open-sourced Swagger API. In the early days, Swagger was mostly used together with a
Swagger UI console that provided an attractive self-documenting API playground.

**This was and still is a great way to visually judge the clarity of an API.**
{: .notice--info}

**Having Swagger UI available to test the API in the development environment makes the development flow developer friendly.**
{: .notice--info}

{% capture fig_img %}
![Swagger UI](/assets/2021-04-02-api-first-done-right/swagger-ui.jpg)
{% endcapture %}
{% capture fig_caption %}
SwaggerUI as an API playground.
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>{{ fig_caption | markdownify | remove: "<p>" | remove: "</p>" }}</figcaption>
</figure>

### Adding SwaggerUI to existing projects

Many projects that retrofittedd Swagger UI to previously existing projects, usually derived the API definition
by scanning the existing source code. Or in the worst case maintaining source code and the Swagger definition independently.

{% capture fig_img %}
![Swagger annotations in code](/assets/2021-04-02-api-first-done-right/swagger-annotations-in-code.jpg)
{% endcapture %}
{% capture fig_caption %}
Swagger/OpenAPI metadata annotations in code (marked in red). These annotations serve no purpose at the application runtime.
@Api* annotations belong to the Swagger 2.0 package (described later).
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>{{ fig_caption | markdownify | remove: "<p>" | remove: "</p>" }}</figcaption>
</figure>

Despite the multiple advantages, the approach where the API definition is just a second class citizen has some
non-obvious shortcomings:

- **Nonneterministic Swagger specification generation**
  
  There are many generator tools each of them providing different output for the same input.
  For example the Maven build-time plugin that I used in the past could not handle custom/complex POST payload
  examples thus limiting the attractiveness of my API. Learning this after I invested days in the development was not a
  pleasant experience.
  
- **The implementation diverging from the Swagger specification**
  
  Having the definition derived from code, sometimes it happens that what you have in your API definition is far away
  from the actual service behavior (for example when you use implicit headers or argument resolvers in Spring MVC).
- **Slower application startup**
  
  Some generators such as SpringFox build the definition at the application startup. Classpath scanning might take some
  time if incorrectly configured.
  
- **Metadata annotations leaking into the business code**
  
  The code is spoiled with additional metadata annotations and delivery artefact is bloated with an additional
  dependency  that provides no additional business value when deployed to production.

Swagger later became what we now know as OpenAPI 3.0, currently there are two incompatible Java annotation packages:
- Swagger 2.0: [https://mvnrepository.com/artifact/io.swagger/swagger-annotations](https://mvnrepository.com/artifact/io.swagger/swagger-annotations)
- OpenAPI 3.0: [https://mvnrepository.com/artifact/io.swagger.core.v3/swagger-annotations](https://mvnrepository.com/artifact/io.swagger.core.v3/swagger-annotations)


## API First

With this approach **the API comes first** and then comes the implementation.
In other words **the source code is no longer the API's source of truth**.

### API First - the mental shift

Developers no longer assume who their client is, whether it is an existing site, a desktop app,
or just a product that has not yet been created.

If for some reason developers decide they don’t like the contract, **changes at the design phase are cheaper by a couple
orders of magnitude** than in the scenario when the contract is defined by an already developed implementation.

**With the API First approach, your OpenAPI file becomes a contract and takes precedence over the code — both server 
(skeleton) and client SDKs are now derived from that contract.**
{: .notice--info}

A typical server development flow:

- Gather requirements and design the API
- Lint the API against company standards/get feedback/review/improve
- Generate code:
  - server skeleton
  - client SDKs 
  - generate stubs/mocks for your clients to start the integration

Advantages (for the business):

- Shorter time to market
- Parallel development flow and shorter feedback loop
  
  Client application/frontend application can be developed independently on the development of the server application.
  {: .notice--info}
- Great potential of building additional value on top of the APIs designed with versatility in mind
- Gaining a competitive advantage by allowing the customers to build their products and tools on top of well designed APIs.

Advantages (for the creator and consumer):

- Infrastructure glue code doesn't have to be written by hand - more time for the business logic development
- Improved API quality and consistency across multiple products of the same family
- No need to maintain dedicated client SDKs parallel to the server code
- Reduced cognitive complexity - when using a generated SDK, from the client point of view,
  remote services can be handled as if they were some local repositories
- Improved developer confidence and satisfaction

## Code generation

Having OpenAPI as a machine-readable API definition made it possible to avoid human-developed API glue code
and it's **costly maintenance** (sometimes done independent for each component and each supported language). 

The tools can be used to generate:
- the **client SDKs** for various languages and language flavours
- **the backend service skeleton**
- **the service mocks** (useful during the client app development)

Code generation is actually the first place where **API First approach unleashes the full power** - it indirectly
enforces the contract continous valibility as any backward incompatible change must start with a highly visible
OpenAPI file change (as opposed to an inconspicuous code modification).
  

### Code generation tools

There are two most popular code generation tools:

- [https://github.com/OpenAPITools/openapi-generator](https://github.com/OpenAPITools/openapi-generator) (**recommended**)
- [https://github.com/swagger-api/swagger-codegen](https://github.com/swagger-api/swagger-codegen) (the original project)

They are both written in Java and support code generation for over 50 different languages and flavors.

The generators can be used:
- from a command line
- as build system plugins (Maven, Gradle)
- as wrappers (NPM tools)

The generator tools define some [common configuration options](https://openapi-generator.tech/docs/configuration/),
additionally each supported language provides a different set of configurable parameters.

A complete of all the generators and the supported options can be found at [https://openapi-generator.tech/docs/generators](https://openapi-generator.tech/docs/generators).
{: .notice--info}

### Code generation in a project lifecycle

Code generation can be done either:

- during the **project build** stage

  Code generation is part of the project build (Maven/Gradle/NPM). Generated files are kept outside version control
  (`target/generated` for Maven or `build/generated` for Gradle), the source is included during the compilation phase.
  This approach allows for greater code generation tuning and simpler CI/CD pipelines at the cost of the build time.

- as **independently released libraries** released using a dedicated pipeline

  Client SDKs are released independently and used as external dependencies (JARs for Java, NPM packages for JavaScript).
  This approach makes project build faster at the cost of limited flexibility.
  In case of the frontend application builds it allows avoiding mixing frontend build technologies with Java tools.

### Generating server skeleton - Spring Boot

Generating server skeleton in a typical Spring Boot application means that you let the generator generate DTOs and the
controller interfaces — all you need to do is to implement these interfaces with concrete controllers.

{% capture fig_img %}
![API Interface](/assets/2021-04-02-api-first-done-right/api-interface.jpg)
{% endcapture %}
{% capture fig_caption %}
Generated controller interface ready to be implemented by a concrete class.
When the API contract changes, so does the (programming) interface.
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>{{ fig_caption | markdownify | remove: "<p>" | remove: "</p>" }}</figcaption>
</figure>

The DTOs and controller interfaces are usually derived at the build time and **kept outside the version control**.
{: .notice--info}

{% capture fig_img %}
![Code generation - Spring](/assets/2021-04-02-api-first-done-right/code-generation-spring.jpg)
{% endcapture %}
{% capture fig_caption %}
A model class generated using a customized model template (added Lombok support).
{% endcapture %}

<figure>
  {{ fig_img | markdownify | remove: "<p>" | remove: "</p>" }}
  <figcaption>{{ fig_caption | markdownify | remove: "<p>" | remove: "</p>" }}</figcaption>
</figure>



Maven plugin configuration from the [https://github.com/piotrpolak/spring-boot-playground](https://github.com/piotrpolak/spring-boot-playground) prject.
Please note `configOptions` for the `spring` language:


```xml
<plugin>
    <groupId>org.openapitools</groupId>
    <artifactId>openapi-generator-maven-plugin</artifactId>
    <version>${openapi-generator-maven-plugin.version}</version>
    <executions>
        <execution>
            <goals>
                <goal>generate</goal>
            </goals>
            <configuration>
                <inputSpec>${project.basedir}/src/main/resources/static/swagger.json</inputSpec>
                <generatorName>spring</generatorName>
                <output>${project.build.directory}/generated-sources/swagger</output>
                <apiPackage>ro.polak.springbootplayground.api</apiPackage>
                <modelPackage>ro.polak.springbootplayground.api.dto</modelPackage>
                <invokerPackage>ro.polak.springbootplayground.api</invokerPackage>
                <modelNameSuffix>Dto</modelNameSuffix>
                <generateSupportingFiles>false</generateSupportingFiles>
                <templateDirectory>${project.basedir}/templates/JavaSpring</templateDirectory>

                <!-- More options at https://openapi-generator.tech/docs/generators/spring -->
                <configOptions>
                    <dateLibrary>java8</dateLibrary>
                    <interfaceOnly>true</interfaceOnly>
                    <library>spring-mvc</library>
                    <performBeanValidation>true</performBeanValidation>
                    <skipDefaultInterface>true</skipDefaultInterface>
                    <useOptional>true</useOptional>
                    <sourceFolder>src/gen/java/main</sourceFolder>
                </configOptions>
            </configuration>
        </execution>
    </executions>
</plugin>
```

#### Code generation shortcomings - Spring Boot

- Explicit `ResponseType` as controller return types
  
  If you like your controllers to return value objects directly or using void methods together `@ResponseStatus`
  annotation — you will have to get used to the new style.
  
- Lack of the support for the [ArgumentResolvers](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/method/support/HandlerMethodArgumentResolver.html)
- [JSR Bean validation](https://beanvalidation.org/1.0/spec/) rules are leaked into the contract details or need
  to be added programmatically
- Code formatting issues and unused package imports
- Code spoiled with metadata `@Api*` annotations belonging to Swagger 2.0 — these come from an
  outdated [swagger-core](https://mvnrepository.com/artifact/io.swagger/swagger-core) dependency (no longer maintained since 2017).
  
  These annotations are only relevant when you use [SpringFox](https://springfox.github.io/springfox/) (out-of-box SwaggerUI integration with application bootstrap
  classpath scanning) but this is definitively useless when you already have a well-crafted
  definition available at your hand.
  

#### Adding JSR validation

- Annotations in OpenAPI definition
  
  The resulting code includes JSR annotations at the field and class levels, however this causes the validation details
  to leak into the OpenAPI specification.
  
- Adding validation rules programmatically

  Hibernate validator allows adding validation rules to any object. The code is characterized by high verbosity and
  limited type safe constructs.

### Generating client SDK for backend-to-backend communication - Java

There are few generators available for client SDK in java, these are build around multiple HTTP libraries, such as:
`jersey`, `jersey2`, `feign`, `okhttp-gson` (default), `retrofit2`, `resttemplate`, `webclient`,
`resteasy`, `vertx`, `google-api-client`, `rest-assured`, `native` (Java 11 only), `microprofile`
  
#### Picking the right generator for Java client SDK

Rules for picking a generator:

- **available project dependencies**

  Whenever possible, pick a generator that does not require any additional dependencies.

- **ease of customization** of the underlying HTTP client

  Based on your needs, pick a generator that uses a client that allows customizing the HTTP client options such as
  timeouts, interceptors and response processors. Clients (with few exceptions) support injecting a preconfigured HTTP client externally.
  
- **consistency with existing HTTP clients** and other SDK clients in the project

  If your Spring project already uses preconfigured HTTP client such as `RestTemplate` or `WebClient` it is probably
  worth reusing it and benefit from the preconfigured connection options and request interceptors.
  There is a chance that the Spring Framework takes care of injecting/forwarding `Correlation-Id` and `Authorization` headers.
  
If the generator (HTTP library) needs to be changed after it has already been used in code, the following challenges
need to be addressed:
- **HTTP client configuration**

  HTTP client configuration and request interceptors are not portable across the different HTTP libraries.
  
- **error handling**

  Each generator (HTTP library) throws different runtime exceptions upon communication issues. You need to review your
  error handlers an controller advices to make sure the new types of exceptions are handled properly.
  
  If your code handles connection exception manually (sometimes found to capture `Resource Not Found`), then you need
  to review all manual exception handling.

  ```java
  try {
    bookstoreApi.getBookById(123);
  } catch(ConnectionException e){
    // This code will have to be changed
    // TODO Capture 404 and rethrow the rest
  }
  ```

  ```java
  try {
    bookstoreApi.getBookById(123);
  } catch(FeignException e){
    // Doing the same as above 
  }
  ```

Ask yourself a question whether you actually need to capture the exception — in many cases those situations are
not recoverable and are globally logged as errors anyway.
{: .notice--warn}

#### Switching from one HTTP library to another

When switching from one library to another, your client code should be fine as long as you don’t add any custom
exception handling. Unfortunately most of the clients throw different types of exceptions (Runtime).

#### Adding/forwarding custom headers

Adding/forwarding custom headers can be obtained in two ways:

- Adding a default headers — this might be suitable for values that are constant over time and shared across all the
  users — for  an API Key:
  ```java
  bookstoreApi.getClient()
    .addDefaultHeader("API-KEY", globalConfig.getApiKey())
  ```
  
  NOTE: calling `addDefaultHeader` after the API client has been initialized will change the state of the shared client
  and might cause concurrency/security issues.
  {: .notice--warning}
  
- Adding headers dynamically — whenever the header values differ from invocation to invocation, the best way to add
  additional headers is to use interceptor mechanism of the underlying libraries.

  ```java
  RestTemplate restTemplate = new RestTemplate();
  restTemplate.addInterceptrs(asList(
    (request, response) ->
      request.getHeaders().add("Custom-Authorization-Header", …))
  );
  BookstoreApi bookstore = new BookstoreApi(restTemplate);
  ```
  **Warning**: your project might already configure the HTTP client bean with some default interceptors/forwarders — this is
  common for adding Correlation-ID.
  {: .notice--warning}

#### Customizing the code templates

Having so many generator options (see possible options for [Spring generator
only](https://openapi-generator.tech/docs/generators/spring/)) implies that the default templates are generic to serve
everyone’s needs.

Some code bloat of the shortcomings mentioned in the previous section can be addressed by **copying and
customizing code generator templates into your project**.

The templates use Mustache as the engine and usually require
one-time customization action. More on templating can be found at [https://openapi-generator.tech/docs/templating](https://openapi-generator.tech/docs/templating)
Things that are not possible to be customized with the templates only can be customized by creating a custom generator
language (see [SpringCodeGen.java](https://github.com/OpenAPITools/openapi-generator/blob/master/modules/openapi-generator/src/main/java/org/openapitools/codegen/languages/SpringCodegen.java)
for reference).

### Generating client SDKs for front-end-to backend communication

Frontend SDK can either be generated during the backend component build (and released as a dedicated NPM package) or during
the frontend application build.

The generated SDK, when combined with TypeScript, can help to spot an incompatible change in the API during build
process rather than during manual tests.

The TypeScript SDKs can either be generated:
- using the NPM wrapper around the OpenAPI tools (mentioned earlier)
  [openapi-generator-cli](https://www.npmjs.com/package/@openapitools/openapi-generator-cli)
- using JavaScript native tool, for example [openapi-typescript-codegen](https://www.npmjs.com/package/@lsongzhi/openapi-typescript-codegen)

Using a JavaScript native tool makes it possible to avoid the need of installing Java on the development and build machines.

A list of alternative generator tools can be found at [https://openapi.tools/#sdk](https://openapi.tools/#sdk).
Pick the best suiting tool based on your needs and FE team preferences.

### Testing using the generated SDK

Integration tests can be written using the SDK to further enhance the API development process. When using this approach,
tests are written using an SDK that is kept outside a version control.

If the existing tests fail to compile against a freshly generated SDK it means the OpenAPI got a backward incompatible change.
{: .notice--warning}

Tests using SDKs should not replace the contract test suits as it is possible to have two compatible SDKs generated for
two slightly different contracts.
{: .notice--warning}

### Importing OpenAPI definition in Postman

A well written OpenAPI definition can easily be imported into Postman. With good request examples and definitions you
can try the happy flows with the minimum adjustments (this might be as simple as providing the correct Authorization
header values).

![Postman](/assets/2021-04-02-api-first-done-right/postman.jpg)

![Postman](/assets/2021-04-02-api-first-done-right/postman2.jpg)

![Postman](/assets/2021-04-02-api-first-done-right/postman3.jpg)

### Mocking your API

With the use of dedicated packages you can transform your rich OpenAPI definition into a realistic mock that can be used
for the frontend application development, independently on the backend services development.

See [openapi-mock-express-middleware](https://www.npmjs.com/package/openapi-mock-express-middleware) or [open-api-mocker](https://www.npmjs.com/package/open-api-mocker) for more details.

A comprehensive comparison of the mocking servers can be found at [https://openapi.tools/#mock](https://openapi.tools/#mock).

## Best practices when maintaining SDK-friendly OpenAPI contracts

See [Best practices when maintaining SDK-friendly OpenAPI contracts]({{ site.baseurl }}{% link _posts/2021-04-08-best-practices-when-maintaining-sdk-friendly-openapi-contracts.markdown %}).

## Additional resources

- [https://openapi.tools/](https://openapi.tools/)
- [https://swagger.io/resources/articles/adopting-an-api-first-approach/](https://swagger.io/resources/articles/adopting-an-api-first-approach/)
- [https://en.wikipedia.org/wiki/Swagger_(software)](https://en.wikipedia.org/wiki/Swagger_(software))
- [https://medium.com/adobetech/three-principles-of-api-first-design-fa6666d9f694](https://medium.com/adobetech/three-principles-of-api-first-design-fa6666d9f694)
- [https://en.wikipedia.org/wiki/OpenAPI_Specification](https://en.wikipedia.org/wiki/OpenAPI_Specification)
- [https://medium.com/better-practices/api-first-software-development-for-modern-organizations-fdbfba9a66d3](https://medium.com/better-practices/api-first-software-development-for-modern-organizations-fdbfba9a66d3)

*[SOAP]: Simple Object Access Protocol
*[REST]: Representational state transfer
*[RPC]: Remote Procedure Call
*[HTTP]: Hypertext Transfer Protocol
*[WSDL]: Web Services Description Language
