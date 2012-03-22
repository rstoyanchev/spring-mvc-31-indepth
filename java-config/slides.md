!SLIDE subsection
# MVC Java Config Design

!SLIDE bullets incremental
# Design goals

* A simple starting point
* Transparency
* Flexibility
* A path from simple to advanced

!SLIDE
# Why does it matter?

!SLIDE
## It's a relatively simple mechanism worth understanding

!SLIDE
## You may use for your own purposes, e.g.
## to design your your own `@EnableXyz`

!SLIDE small
# A simple starting point

	@@@ java

        @EnableWebMvc
        @Configuration
        public class WebConfig {

        }

!SLIDE
## No base class
## No Spring MVC beans in sight

!SLIDE bullets incremental
# What it provides

* @MVC request processing
* `ConversionService`
* Global JSR-303 validator
* `HttpMessageConverter` registrations
* A few others

!SLIDE bullets incremental
# What can be customized

* Implement `WebMvcConfigurer`
* Or extend `WebMvcConfigurerAdapter`

!SLIDE
## Still no Spring beans
## Implement an interface and magic happens

!SLIDE
# Demo
## <a href="https://github.com/SpringSource/spring-framework-issues/tree/master/SPR-0000-war-java">SPR-0000-war-java</a>

!SLIDE
## No Spring beans
## Implement an interface and magic happens

!SLIDE
# How does it work?

!SLIDE small

	@@@ java

        @EnableWebMvc  // <-- What's behind?
        @Configuration
        public class WebConfig {

        }

!SLIDE small transition=scrollLeft
# `@EnableWebMvc`

    @@@ java

        @Retention(RetentionPolicy.RUNTIME)
        @Import(DelegatingWebMvcConfiguration.class)
        @Target(ElementType.TYPE)
        public @interface EnableWebMvc {

        }

!SLIDE small transition=fade
# `@EnableWebMvc`

    @@@ java


        @Import(DelegatingWebMvcConfiguration.class)

        public @interface EnableWebMvc {

        }

!SLIDE
## The configuration is imported

!SLIDE bullets incremental small
# `WebMvcConfigurationSupport`

* Contains the imported configuration
* Uses `@Bean` methods
* Just like your configuration
* Easy to read

!SLIDE
# Demo

!SLIDE
## How does importing the configuration allow for customizations?

!SLIDE
## Recall that we
## implement an interface and magic happens


!SLIDE smaller bullets incremental
# `DelegatingWebMvcConfigurationSupport`

* A sub-class of `WebMvcConfigurationSupport`
* "Detects" `WebMvcConfigurer` implementations
* Via `@Autowired`
* Delegates to detected instances to customize the config

!SLIDE
## However at any point you can switch
## to extending directly from
## **`WebMvcConfigurationSupport`**

!SLIDE bullets incremental
## Extending directly from `WebMvcConfigurationSupport`

* Remove `@EnableWebMvc`
* i.e. don't import
* Extend `WebMvcConfigurationSupport`
* Override `@Bean` and other available methods


