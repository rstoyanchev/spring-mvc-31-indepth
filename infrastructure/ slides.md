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
# Old @MVC
# Request Mapping Process

* Two stages
* `HandlerMapping` selects controller
* <span style="color:blue">Perhaps not from annotations!</span>
* `HandlerAdapter` narrows down the method

!SLIDE
## Designed to work with
## different `HandlerMapping` types
## E.g.
## `SimpleUrlHandlerMapping` <br> and others

!SLIDE bullets incremental
# Sounds reasonable but..

* Controller is chosen mainly based on URL `+` type-level `@RequestMapping` attributes
* Method-level `@RequestMapping` attributes deferred to `HandlerAdapter`
* Splits decision making process

!SLIDE smaller
# Back to the code

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
# Back to the code

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller  // (1) Controller selected based on "/foo"
    public FooController {
    
    

    
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE smaller
# Back to the code

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller  // (1) Controller selected based on "/foo"
    public FooController {
    
      // (2) Only to find later that method doesn't match
      

      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE smaller
# Back to the code

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller  // (1) Controller selected based on "/foo"
    public FooController {
    
      // (2) Only to find later that method doesn't match
      // (3) Harder to reason at this stage about the error

      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    
    }

!SLIDE bullets incremental
# Other limitations of prior approach

* One URL per controller   
* Controller method not known to `HandlerInterceptor`
* Ditto for `HandlerExceptionResolver`
* Harder to reason about the process

!SLIDE bullets incremental
# New @MVC

* `HandlerMethod` abstraction
* Think of methods as endpoints
* Each endpoint must be uniquely mapped!

!SLIDE bullets incremental
# A single point of decision

* `HandlerMapping` selects a `HandlerMethod`
* Not a controller
* `HandlerAdapter` invokes selected method
* Doesn't know how it's chosen

!SLIDE smaller
# The code re-visited with 3.1

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
# The code re-visited with 3.1

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    
      // A single point of decision

      
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    }

!SLIDE smaller
# The code re-visited with 3.1

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    
      // A single point of decision
      // Result is 406 (NOT_ACCEPTABLE)
            
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      headers="Accept=text/plain")
      public @ResponseBody Foo getValue() {
      
      }
    
    }

!SLIDE smaller
# The code re-visited with 3.1

    @@@ java

	// GET /foo Accept=text/html
    
    @Controller
    public FooController {
    
      // A single point of decision
      // Result is 406 (NOT_ACCEPTABLE)
            
      @RequestMapping(value = "/foo",
                      method=RequestMethod.GET, 
                      produces="text/plain") // <-- produces
      public @ResponseBody Foo getValue() {
      
      }
    
    }

!SLIDE bullets incremental
# Other limitations lifted

* Same URL processing can be <br> split across controllers
* Controller method known to `HandlerInterceptor`, `HandlerExceptionResolver`

!SLIDE
# Demo
## <a href="https://github.com/rstoyanchev/spring-mvc-31-demo">spring-mvc-31-demo</a>
<br>

!SLIDE
## Incidentally after the 3.1 release we had this ticket

!SLIDE bullets incremental
# <a href="https://jira.springsource.org/browse/SPR-9063">SPR-9063</a>

* The following code worked in 3.0
* Treated as "ambiguous mapping" in 3.1

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
	
	// Can you guess why {GET, POST} are on the type level?
	
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

!SLIDE smaller transition=fade
# The code with the new @MVC
## Just remove the type-level HTTP method mapping

	@@@ java

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

* Automatic upgrade with the <br> MVC namespace or MVC Java config
* You must update configuration otherwise
* We recommended you do so
* No more new features on old @MVC

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

!SLIDE
# Demo



  