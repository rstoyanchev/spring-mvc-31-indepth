
!SLIDE subsection
# `RedirectAttributes`

!SLIDE smaller
# Example

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

* The `Model` good for rendering views
* Not so much for redirecting
* Prefer `RedirectAttributes` when redirecting for better control

!SLIDE smaller
# Example

    @@@ java

    @RequestMapping(method=POST)
    public String save(Entity entity, Errors errors,
                       Model model,
                       RedirectAttributes redirectAttrs){


        // Use Model if not redirecting (e.g. Ajax request)


        // Use RedirectAttrs otherwise

    }

!SLIDE bullets incremental
# Enable use of `RedirectAttributes` for all redirects

* For example when method signature does not contain `RedirectAttributes`
* `RequestMappingHandlerAdapter` property
* `setIgnoreDefaultModelOnRedirect`

!SLIDE bullets incremental
# Flash attributes

* Redirect URL is the intended recipient 
* But what if another request comes in meanwhile?
* Not uncommon with Ajax requests
* Potentially delivered to wrong recipient

!SLIDE bullets incremental
# `FlashMap`

* Contains properties with target URL recipient and params
* Auto-populated by `RedirectView`
* After the redirect request matched to `FlashMap` in `FlashMapManager`



