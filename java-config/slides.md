!SLIDE subsection
# MVC Java Config Design

!SLIDE
# Design goal

!SLIDE incremental
## Replicate the benefits of
## the MVC XML namespace in Java

!SLIDE incremental
## minus the downsides

!SLIDE incremental bullets
# MVC namespace

* On-ramp for Spring MVC applications
* Higher-level config language
* Succinct
* Little knowledge required

!SLIDE bullets incremental
# But also..

* Config not easy to see
* No path from simple to advanced
* Black box

!SLIDE bullets incremental
# Even more confusing to..

* Copy & paste config snippets
* When looking to solve a problem
* How does it relate to the namespace?

!SLIDE
# Lesson learned:

## Transparency is key for Spring MVC config!

!SLIDE
## And so is flexibility

!SLIDE bullets incremental small
# `WebMvcConfigurationSupport`

* The central MVC Java config class
* Config comparable to that of the MVC namespace
* But using `@Bean` methods
* Easy to read

!SLIDE bullets incremental small
# How to make use of it?

* One obvious choice..
* Extend `WebMvcConfigurationSupport`
* Override `@Bean` methods

!SLIDE bullets incremental
## But how to replicate the MVC namespace experience?

* A simple starting point
* Little knowledge required
* Higher-level config language

!SLIDE small
# Simple starting point

	@@@ java

        @EnableWebMvc
        @Configuration
        public class WebConfig {

        }


!SLIDE small
# Simple starting point

	@@@ java

        @EnableWebMvc   // <-- What's behind ?
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
## i.e. 
## import rather than extend the configuration

!SLIDE 
## What about config customizations and
## a higher-level config language?

!SLIDE smaller
# Customizations

    @@@ java

    @EnableWebMvc
    @Configuration
    public class WebConfig implements WebMvcConfigurer {









    }

!SLIDE smaller
# Customizations

    @@@ java

    @EnableWebMvc
    @Configuration
    public class WebConfig implements WebMvcConfigurerAdapter {









    }

!SLIDE smaller
# Customizations

	@@@ java

    @EnableWebMvc
    @Configuration
    public class WebConfig implements WebMvcConfigurerAdapter {

        @Override
        public void addInterceptors(InterceptorRegistry reg){

        }
 
 
        // More options in WebMvcConfigurer..

    }

!SLIDE
## still no Spring MVC beans in our base class!
### (config imported instead)

!SLIDE smaller bullets incremental
# How does it work?

* You **import** `DelegatingWebMvcConfigurationSupport`
* which extends `WebMvcConfigurationSupport`
* "Detects" implementations of `WebMvcConfigurer`
* (via `@Autowired`)
* Delegates to all to customize the config

!SLIDE
## Still at any point you can switch
## to extending directly from
## **`WebMvcConfigurationSupport`**

