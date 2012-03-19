!SLIDE subsection
# The New <br> @MVC Support Classes

!SLIDE
# Old 
## `DefaultAnnotationHandlerMapping`, `AnnotationMethodHandlerAdapter`, `AnnotationMethodExceptionResolver`
<br>
# New
## `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, `ExceptionHandlerExceptionResolver`

!SLIDE
# Why the re-write?

!SLIDE bullets incremental
# General Background

* JIRA tickets for more flexibility
* A few functional limitations
* A few tricky request mapping semantics 

!SLIDE
# The ticket that
# happened to tip the balance

!SLIDE bullets incremental small
# <a href="https://jira.springsource.org/browse/SPR-7345">SPR-7345</a>

* Relatively insignificant
* Not many comments
* Nevertheless worth a closer look
* A representative example

!SLIDE bullets incremental small
# The issue

* 404 expected
* 405 (`METHOD_NOT_SUPPORTED`) returned
* ?

!SLIDE smaller
# The code

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    
    
    
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE smaller
# The code

    @@@ java

	// GET /foo Accept=text/html	<- Accept header
    
    @Controller
    public FooController {
    
    
    
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")  // No match
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE bullets incremental
# Request Mapping Process
## Old @MVC

* Two-stage decision
* `HandlerMapping` selects controller
* <span style="color:blue">Perhaps not from annotations!</span>
* `HandlerAdapter` selects the method

!SLIDE bullets incremental
# Sounds reasonable but..

* Controller is chosen mainly based on URL `+` type-level `@RequestMapping` attributes
* Method-level `@RequestMapping` attributes deferred to `HandlerAdapter`
* Splits decision making process

!SLIDE smaller
# Back to our example

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    
    
    
    
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE smaller
# Back to our example

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller  // (1) Controller selected
    public FooController {
    
    

    
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE smaller
# Back to our example

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller  // (1) Controller selected
    public FooController {
    
      // (2) Only to find that method doesn't match
      

      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE smaller
# Back to our example

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller  // (1) Controller selected
    public FooController {
    
      // (2) Only to find that method doesn't match
      // (3) Harder to reason about the error at this point

      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE bullets incremental
# Other drawbacks

* One URL per controller   
* Selected method not exposed to `HandlerInterceptor`
* Nor to `HandlerExceptionResolver`
* Harder to reason

!SLIDE bullets incremental
# New @MVC

* `HandlerMethod` abstraction
* Think of methods as endpoints
* Each endpoint must be uniquely mapped!

!SLIDE bullets incremental
# A single point of decision

* `HandlerMapping` selects a `HandlerMethod`
* Not a controller
* `HandlerAdapter` simply invokes the method
* Not involved in selecting it

!SLIDE smaller
# The example re-visited in 3.1

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    


      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    }

!SLIDE smaller
# The example re-visited in 3.1

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    
    
      
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      produces="text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    }

!SLIDE smaller
# The example re-visited in 3.1

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    
      // Results in 406 (NOT_ACCEPTABLE)
            
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      produces="text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    }

!SLIDE bullets incremental
# Other limitations lifted

* Same URL processing can be split across controllers
* _(it's just method endpoints)_
* Selected method exposed to `HandlerInterceptor`, `HandlerExceptionResolver`

!SLIDE
## Incidentally after the 3.1 release we had this ticket

!SLIDE bullets incremental
# <a href="https://jira.springsource.org/browse/SPR-9063">SPR-9063</a>

* The following code worked in 3.0
* Treated as "Ambiguous mapping" in 3.1
* _(code relied on the two-stage process)_

!SLIDE smaller
# The code

	@@@ java
	
	
	
	@Controller
	@RequestMapping(value = "/foo",
	    method = {RequestMethod.GET, RequestMethod.POST })
	public class FooController {
	
	    @RequestMapping(method = RequestMethod.GET)
	    public String index() {
	        return "form";
	    }
	
	    @RequestMapping(method = RequestMethod.POST)
	    public String submit() {
	        return "success";
	    }
	
	}

!SLIDE smaller
# The code

	@@@ java
	
	// Note the extra type-level HTTP methods!
	
	@Controller
	@RequestMapping(value = "/foo",
	    method = {RequestMethod.GET, RequestMethod.POST })
	public class FooController {
	
	    @RequestMapping(method = RequestMethod.GET)
	    public String index() {
	        return "form";
	    }
	
	    @RequestMapping(method = RequestMethod.POST)
	    public String submit() {
	        return "success";
	    }
	
	}

!SLIDE smaller
# The code revisited in the new @MVC

	@@@ java
	

	
	@Controller
	@RequestMapping(value = "/foo",
	    method = {RequestMethod.GET, RequestMethod.POST })
	public class FooController {
	
	    @RequestMapping(method = RequestMethod.GET)
	    public String index() {
	        return "form";
	    }
	
	    @RequestMapping(method = RequestMethod.POST)
	    public String submit() {
	        return "success";
	    }
	
	}

!SLIDE smaller
# The code revisited in the new @MVC

	@@@ java
	
	// Just remove the type-level HTTP methods!
	

	@Controller
	@RequestMapping(value = "/foo")
	public class FooController {
	
	    @RequestMapping(method = RequestMethod.GET)
	    public String index() {
	        return "form";
	    }
	
	    @RequestMapping(method = RequestMethod.POST)
	    public String submit() {
	        return "success";
	    }
	
	}

!SLIDE
# Is it not a regression?

!SLIDE bullets incremental
# The upgrade path

* Automatical upgrade if using `<mvc:annotation-driven>` or `@EnableWebMvc`
* Must update your configuration otherwise
* We recommended you do so
* Old @MVC won't get any new features

!SLIDE bullets incremental
## `HandlerMethod`:
## An abstraction for various controller methods

* `@RequestMapping`
* `@ModelAttribute`
* `@InitBinder`
* `@ExceptionHandler`

!SLIDE smaller
# How different method arguments are supported

    @@@ java


	public interface HandlerMethodArgumentResolver {
	
      boolean supportsParameter(MethodParameter parameter);
	
      Object resolveArgument(
		  MethodParameter parameter, 
		  ModelAndViewContainer mavContainer,
		  NativeWebRequest webRequest, 
		  WebDataBinderFactory binderFactory) throws Exception;
		  
	}

!SLIDE
## Every built-in method argument has its own `HandlerMethodArgumentResolver`

!SLIDE smaller
# How different return value types are supported

    @@@ java


	public interface HandlerMethodReturnValueHandler {
	
      boolean supportsReturnType(MethodParameter returnType);
	
      void handleReturnValue(
        Object returnValue,
		MethodParameter returnType,
		ModelAndViewContainer mavContainer,
		NativeWebRequest webRequest) throws Exception;
		
	}

!SLIDE 
## Every supported return value has its own `HandlerMethodReturnValueHandler`

!SLIDE incremental bullets
# The benefits

* More flexible
* More customizable
* Better flexible method signature support




  