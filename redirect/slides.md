
!SLIDE subsection
# Redirect Attributes

!SLIDE
## This stack overflow <a href="http://stackoverflow.com/questions/8972663/do-session-attributes-show-in-url-when-handling-post-redirect-in-spring-mvc-3">post</a> describes the motivation

!SLIDE smaller
# `RedirectAttributes` Example

    @@@ java

        @RequestMapping(method=POST)
        public String save(Entity entity, Errors errors,
                           RedirectAttributes redirectAttrs){

            // ...

            redirectAttrs.addAttribute("id", entity.getId);
            redirectAttrs.addAttribute("date", new Date());

            // ...

            redirectAttrs.addFlashAttribute("message", "Yay!");

            return "redirect:/someUrl";
        }

!SLIDE bullets incremental
# Why not just use the `Model`?

* The `Model` has the right content for rendering
* Not so much for redirecting
* It's usually a different set of attributes
* Prefer `RedirectAttributes` when redirecting for better control

!SLIDE smaller
# This is possible

    @@@ java

    @RequestMapping(method=POST)
    public String save(Entity entity, Errors errors,
                       Model model,
                       RedirectAttributes redirectAttrs){


        // Use Model if not redirecting (e.g. Ajax request)


        // Use RedirectAttrs otherwise

    }

!SLIDE
# Demo
## <a href="https://github.com/SpringSource/spring-mvc-showcase/blob/master/src/main/java/org/springframework/samples/mvc/form/FormController.java">FormController.java</a>
## <a href="https://github.com/SpringSource/spring-mvc-showcase">spring-mvc-showcase</a>

!SLIDE bullets incremental small
# How to enable use of `RedirectAttributes` for all redirects

* For example when method signature does not contain `RedirectAttributes`

!SLIDE smaller
# Example

    @@@ java

    @RequestMapping(method=POST)
    public String save(Entity e, Errors errors, Model model){

		// ...

		return "redirect:/someUrl";
    }


!SLIDE bullets incremental small
# Use `RedirectAttributes` for all redirects

* `RequestMappingHandlerAdapter` property
* `setIgnoreDefaultModelOnRedirect`
    
!SLIDE bullets incremental small
# Flash attributes

* Redirect URL is the intended recipient 
* But what if another request comes in meanwhile?
* Not so unlikely with Ajax requests
* Potentially not consumed by intended recipient

!SLIDE bullets incremental small
# `FlashMap`

* Contains properties with target URL + params
* Auto-populated by `RedirectView`
* After the redirect `FlashMapManager` tries to match request to `FlashMap` instances



