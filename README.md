# Aspect Ratio Custom Properties Provider

A helper script that will enhance the browser's CSS styling capabilities by providing JS-calculated custom properties which allow for the creation of calc and clamp CSS functions that would otherwise not be possible with the way CSS currently works.

Copyright 2022 Stefan Winkler (<https://github.com/Sigma-90>) - The MIT License provided in the same folder as this README file applies.
  
## JS Usage

This script can be used in various different ways, depending on how you intend to use it:

### When prioritizing page content over potential layout shifts

If the risk of a brief moment, where fallback styles apply before everything is ready, is acceptable for your use case, you may load *aspect-ratio-custom-properties-provider.min.js* asynchronously (either in the page *&lt;head&gt;* or at the end of the *&lt;body&gt;*) and then let it self-initialize by simply adding the class "auto-init-aspect-ratio-c-props-provider" to the *&lt;body&gt;* tag.
The *&lt;body&gt;* tag can also receive two custom attributes:

* `data-aspect-ratio-event-detection-buffer-time="20"` : The number provided here will be the amount of milliseconds all resizing event signals will be suppressed after the first one was received. If this attribute is not set, a default of 25ms will be used.
* `data-aspect-ratio-square-detection-tolerance="0.110"` : The number provided here will be the tolerance value for determining when the viewport's aspect ratio is almost squared. The higher the value (the accepted maximum is 0.999) the more rectangular the viewport shape can be while the effect still toggles, the lower the closer you have to be to a square. If this was 0, queries would only ever work if the viewport width and height are exactly the same.

### When different square detection tolerance values are needed for different pages in your website

For this to work, the CSS rule for the definitions of the custom properties has the be generated by the script contained in *aspect-ratio-custom-property-definitions-generator.min.js* instead of being pre-defined in your own CSS files. Wherever you put the *&lt;script&gt;* tag to load that file, it should be placed before the *&lt;script&gt;* tag that loads the main file *aspect-ratio-custom-properties-provider.min.js*, preferably right before it. In order to specify the tolerance value you can either run your own calculations and then pass it to the initialization function `initializeAspectRatioCustomPropertiesProvider` when you call it in your own JS files, or you can use the self-initialization method described in the previous section by adding not only the class name "auto-init-aspect-ratio-c-props-provider" to the *&lt;body&gt;* tag but also the custom attribute "data-aspect-ratio-square-detection-tolerance" and providing a valid value for it. Of course this means that your main CSS files should not contain conflicting rules, so proper fallbacks are required. Because the main script will add the class "aspect-ratio-c-props-available" to the *&lt;body&gt;* tag once everything has been initialized, all rules utilizing the custom properties should therefore be defined like this:

```css
  .my-selector {
    /* No-JS fallback rules */
    ...
  }
  .aspect-ratio-c-props-available .my-selector {
    /* Actual rules using aspect ratio clamp queries */
    ...
  }
```

### When layout shifts should be avoided as much as possible

Place the *&lt;script&gt;* tag that loads the main file *aspect-ratio-custom-properties-provider.min.js* in your document head to ensure that it loads as soon as possible. Even better would be to output the file's contents directly inside a *&lt;script&gt;* block so no additional file has to be loaded in the first place. It must not be loaded asynchronously in this case, otherwise the functions contained in the script file might not be available when you attempt to call them.
  
Then call the initialization function `initializeAspectRatioCustomPropertiesProvider`, either in another JS file that is referenced in a *&lt;script&gt;* tag located as close as possible after the one referencing or containing the main script, or - again, preferably - directly inside a script block. In case you went with the recommendation of using a script block with the main script file's content inside, it should be the same one.

Copy the CSS block from the [**CSS Usage**](#css-usage) section of this README into your main CSS file for your website - as high up in the cascade as possible, ideally right below any @font-face and @import declarations. This way you also will never have to load *aspect-ratio-custom-property-definitions-generator.min.js*, which saves some traffic.
  
This way you will have optimized the chances of the script being executed before the domReady event fires, thus preventing layout shifts. These can happen when loaded CSS rules are rendering things initially incorrect, because the CSS custom properties used in the style definitions have not been generated yet. Setting it up this way also eliminates the need for most CSS fallbacks (except for those occasions when JS execution fails), because the recommended CSS black already defines fallback handling in case the expected custom properties set by the script are not present.
  
### Init Params  
  
The initialization function can take two optional parameters:

* **iDebouncingDetectionBufferTimeMs** (integer):

  The amount of milliseconds the debouncing logic will stop detecting resizing event signals after the first one was fired. Only when no resizing happened for the amount of milliseconds specified with this parameter will the custom properties and thus all their dependent dynamic values be recalculated.

  When left out, a default fallback of 25ms will apply. So, if you want to keep the defaults but need to pass in the second or second and third parameter, either provide 25 or something falsy as the first one.

* **bGenerateSquareDetectionBaseStyling** (boolean):

  If true, the function will generate a style element inside the current document head that contains basic custom property definitions needed for being able to write CSS clamp queries. You will find more infos on these in the CSS chapter of this file.

  If you are planning to add such a definition to your main CSS file instead - which is highly recommended by the way, unless different tolerance values are required for different pages - you can just leave this out and call the initialization function without any parameters.

* **fSquareDetectionToleranceWindow** (floating point number between 0 and 1):

  Only applies if *bGenerateSquareDetectionBaseStyling* has been provided with a truthy value. This will define the tolerance value of the generated CSS calculation. Expected is a floating point number between 0 and 1.

  This controls how close the aspect ratio can get to a perfect square before the threshold calculation result flips from a negative to a positive value and vice versa.

  If nothing is provided here, a default value 0f 0.125 is applied, which was established as a good best-practice value through various experiments.

### Output / Core Functionality

When initialized, the script will fill the style attribute of the root &lt;html&gt; tag with 12 custom property definitions and update the last 10 of them every time the viewport size changes. These are:

* **--ratio-min-multiplier**:
  A fixed value set by JS that overrides a fixed fallback value defined in the CSS rules. This ensures that content will not be rendered wrong when the user has JavaScript disabled or errors in other scripts on the page break JS execution.
  
* **--ratio-max-multiplier**:
  See above for what it is and see below under *CSS usage* what it is actually used for.
  All values below will be updated when the viewport size changes for any reason.
  
* **--vh**:
  A multiplier for CSS calc functions, the value will always represent 1% of the current viewport height in pixels, which can be very helpful in certain cases. It also has the benefit of being more accurate towards the actual height of the visible area on mobile phones: Check out  the CSS Tricks article linked at the bottom of this file for more info on that matter.
  
* **--vw**:
  Same as above, but for the viewport width instead of the height.
  
* **--vh-net**:
  This will not return 1% of the entire viewport height but 1% of the net value of it that represents the actual size of the viewport content by deducting the height of potentially displayed horizontal scroll bars from the full height value. Of course the value will be the same as --vh when there is no horizontal scroll bar rendered on your page, so if you are sure that this will never be the case, it is perfectly fine to always use just *--vh* instead.
  
* **--vw-net**:
  Analogous to the ones above, this will take on the value of 1% of the net width of the viewport, ie. the actual width of the visible content area, obtained by deducting the width of the currently displayed vertical scroll bar from the full viewport width. Again, in case a page contains so little content that no scroll bar appears, the value will of course be the same as *--vw*, but given the fact that web pages with such little content are very rare, it would always be better to take scroll bar widths into consideration.

* **--sr**:
  The current viewport screen ratio in the traditional definition of width by height, so it will be 1 for perfect squares, smaller for viewports in portrait view and larger for viewports in landscape view.
  
* **--ar**:
  The current viewport aspect ratio, independent from portrait or landscape mode, as the larger value will always be divided by the smaller one, so the numeric result will never be smaller than 1.
  One practical example:
  calc((var(--sr) - 0.0001 - var(--ar)) * 100000) will result in a large negative value when the screen is in landscape mode and a large positive one when in portrait mode, which can be used with a clamp function to apply one of two CSS values based on the current viewport orientation.
  
* **--sr-net**:
  The screen ratio of the actual net viewport content after deducting the dimensions of any scrollbars on the edges of the viewport from the full viewport dimensions.
  
* **--ar-net**:
  Same as above, just for the aspect ratio instead of the screen ratio.
  
* **--sb-wd**:
  The width of the currently displayed vertical scrollbar. Will be 0 in case no scrollbar is rendered.

* **--sb-ht**:
  The height of the currently displayed horizontal scrollbar. Will be 0 in case no scrollbar is rendered.
  These last two were added for general utility purposes. They do not contribute to any core functionality of this script, but their values had to be determined anyway, so it made sense to then also just expose them to CSS in case a UI developer needs to have access to them for certain use cases. Not everyone might need them but for those who do, their inclusion will ensure that they won't have to use a second script to obtain them.

## CSS Usage

  The generated values listed above are by themselves already very useful, because they are exposing values usable in CSS calc functions that would otherwise be unobtainable.
  
  --vh for example can be used on mobile phones to calculate with the actual height of the viewport, because using just 100vh does not take the address bar into account. --net-vh allows for the same on desktop, because it deducts the size of currently rendered scrollbars from the actual viewport calculation.

  But of course the main intended use case is to detect the current viewport aspect ratio (hence the name of the script file and repository) and to enable that, the script is capable of generating the necessary CSS rule set definition for this if need be.
  
  However, it is recommended to include that definition in your website's main CSS file instead, unless different tolerance values are required for different pages (special treatment for landing pages, for example).
  
  It would have to look like this then:
  
  ```css
      :root {
        --ratio-multiplier-min: var(--ratio-min-multiplier, 1rem);
        --ratio-multiplier-max: var(--ratio-max-multiplier, -1rem);
        --aspect-ratio: var(--ar, 1vh);
        --screen-ratio: var(--sr, 1vh);
        --net-aspect-ratio: var(--ar-net, 1vh);
        --net-screen-ratio: var(--sr-net, 1vh);
        --ratio-calculation-tolerance: 0.125;
        --aspect-ratio-threshold: calc((var(--ratio-calculation-tolerance) + 1 - var(--aspect-ratio)) * 100000);
        --screen-ratio-threshold: calc((var(--ratio-calculation-tolerance) + 1 - var(--screen-ratio)) * 100000);
        --net-aspect-ratio-threshold: calc((var(--ratio-calculation-tolerance) + 1 - var(--net-aspect-ratio)) * 100000);
        --net-screen-ratio-threshold: calc((var(--ratio-calculation-tolerance) + 1 - var(--net-screen-ratio)) * 100000);
        --clamp-query-select-max-when-full-viewport-is-squared: calc(var(--aspect-ratio-threshold) * var(--ratio-multiplier-max));
        --clamp-query-select-min-when-full-viewport-is-squared: calc(var(--aspect-ratio-threshold) * var(--ratio-multiplier-min));
        --clamp-query-select-max-when-full-viewport-is-portrait: calc(var(--screen-ratio-threshold) * var(--ratio-multiplier-max));
        --clamp-query-select-min-when-full-viewport-is-portrait: calc(var(--screen-ratio-threshold) * var(--ratio-multiplier-min));
        --clamp-query-select-min-when-full-viewport-is-landscape: var(--clamp-query-select-max-when-full-viewport-is-portrait);
        --clamp-query-select-max-when-full-viewport-is-landscape: var(--clamp-query-select-min-when-full-viewport-is-portrait);
        --clamp-query-select-max-when-viewport-content-is-squared: calc(var(--net-aspect-ratio-threshold) * var(--ratio-multiplier-max));
        --clamp-query-select-min-when-viewport-content-is-squared: calc(var(--net-aspect-ratio-threshold) * var(--ratio-multiplier-min));
        --clamp-query-select-max-when-viewport-content-is-portrait: calc(var(--net-screen-ratio-threshold) * var(--ratio-multiplier-max));
        --clamp-query-select-min-when-viewport-content-is-portrait: calc(var(--net-screen-ratio-threshold) * var(--ratio-multiplier-min));
        --clamp-query-select-min-when-viewport-content-is-landscape: var(--clamp-query-select-max-when-viewport-content-is-portrait);
        --clamp-query-select-max-when-viewport-content-is-landscape: var(--clamp-query-select-min-when-viewport-content-is-portrait);
        --scrollbar-width: var(--sb-wd, 0px);
        --scrollbar-height: var(--sb-ht, 0px);
      }
  ```
  
  Of course it can also be shortened a bit, if the results of each intermediary calculation step are not needed for other purposes. But treat this shortened notation with caution, it becomes a lot less readable and thus makes it harder to apply fine-tuning by adjusting the tolerance threshold value. It also performs more calculations than the default definition and is thus worse for performance. But for the sake of completeness, here it is anyway:
  
  ```css
    :root {
      --clamp-query-select-max-when-full-viewport-is-squared: calc(calc((1.125 - var(--ar, 1)) * 100000) * var(--ratio-max-multiplier, 1rem));
      --clamp-query-select-min-when-full-viewport-is-squared: calc(calc((1.125 - var(--ar, 1)) * 100000) * var(--ratio-min-multiplier, 1rem));
      --clamp-query-select-max-when-full-viewport-is-portrait: calc(calc((1.125 - var(--sr, 1)) * 100000) * var(--ratio-max-multiplier, 1rem));
      --clamp-query-select-min-when-full-viewport-is-portrait: calc(calc((1.125 - var(--sr, 1)) * 100000) * var(--ratio-min-multiplier, 1rem));
      --clamp-query-select-min-when-full-viewport-is-landscape: var(--clamp-query-select-max-when-full-viewport-is-viewport-content-is-portrait);
      --clamp-query-select-max-when-full-viewport-is-landscape: var(--clamp-query-select-min-when-full-viewport-is-viewport-content-is-portrait);
      --clamp-query-select-max-when-viewport-content-is-squared: calc(calc((1.125 - var(--ar-net, 1)) * 100000) * var(--ratio-max-multiplier, 1rem));
      --clamp-query-select-min-when-viewport-content-is-squared: calc(calc((1.125 - var(--ar-net, 1)) * 100000) * var(--ratio-min-multiplier, 1rem));
      --clamp-query-select-max-when-viewport-content-is-portrait: calc(calc((1.125 - var(--sr-net, 1)) * 100000) * var(--ratio-max-multiplier, 1rem));
      --clamp-query-select-min-when-viewport-content-is-portrait: calc(calc((1.125 - var(--sr-net, 1)) * 100000) * var(--ratio-min-multiplier, 1rem));
      --clamp-query-select-min-when-viewport-content-is-landscape: var(--clamp-query-select-max-when-viewport-content-is-viewport-content-is-portrait);
      --clamp-query-select-max-when-viewport-content-is-landscape: var(--clamp-query-select-min-when-viewport-content-is-viewport-content-is-portrait);
      --scrollbar-width: var(--sb-wd, 0px);
      --scrollbar-height: var(--sb-ht, 0px);
    }
  ```
  
  The multipliers (located in the first two rows of the non-shortened version and inlined in the calculations of the shortened version) are there to provide useful fallback values in case JS execution doesn't happen (either because it is disabled in the browser or other scripts on the page cause a severe error that halts all further JS processing) and thus the custom properties will never be generated. The second parameter in the var getter function defines the fallback and by always setting the multiplier to the opposite of what it should be when script execution works, we ensure that everything looks as intended by default. Otherwise, the styling applying to the defined case of a query custom property (which cannot be detected without scripting) would apply at all times instead of only in the case it was defined for.

  Example: Reducing the size of a banner image when the screen gets close to being squared to make more room for text would look like this when all values are resolved: `height: clamp(25vh, calc(12500rem * var(--ratio-multiplier-min)), 50vh);` Since the first part of the  calculation will always be that fixed number due to no calculations taking place, the negative multiplier needed for the query to work would in this case turn the 12500rem negative and thus the banner would always be shown with the reduced size of 25vh instead of only when the aspect ratio is almost squared. So, by inverting the fallbacks we ensure that the default styling always applies and the special case then simply cannot be shown without JS.

  This block of CSS should be placed very high up in the cascade, preferably right after any @-rules like imports and font definitions.
  
  The --clamp-query-* custom properties can then be used like in this example:
  
  ```css
    .some-class {
        font-size: clamp(min(8vh,2vw), var(--clamp-query-select-max-when-viewport-content-is-squared), 4rem); 
    }
  ```

  *Note: Hopefully you did not become a bit confused because in the example the first argument is not a plain value but another CSS function call, that was just put in there to illustrate that these clamp functions can be built in a lot more complex manner than one might expect at first. Always keep thinking about new possibilities.*

  What this example rule will do is the following:

  When the viewport aspect ratio is almost squared (within the defined tolerance), *--clamp-query-select-max-when-squared* will become a large positive number and thus the third argument of the clamp function (the allowed maximum) will be used, otherwise *--clamp-query-select-max-when-squared* will be a very large negative number and thus the first argument (the allowed minimum) is applied.

  At least if suitable values have been selected for the minimum and maximum: If the maximum value turns out to be smaller than the minimum, unforeseen effects are bound to happen, like expecting the value on the right to be used, but since it undercuts the allowed minimum, the left one will always be picked. Of course being aware of this opens up some interesting possibilities that might actually be put to some use, especially when combined with some more usages of the calc, min, or max functions, so feel free to experiment, as long as you keep the potential pitfalls in the back of your mind.

  The other calculated values work in the same way and will do what their respective names suggest, when applied properly as the middle parameter to a clamp function. So, these dynamic values basically turn the clamp function into an inline media query for the screen aspect ratio, something that is not possible with regular CSS alone.

  You can learn more about clamp pseudo queries in these two brilliant articles:

  Smashing Magazine: <https://www.smashingmagazine.com/2022/08/fluid-sizing-multiple-media-queries/>

  CSS Tricks: <https://css-tricks.com/linearly-scale-font-size-with-css-clamp-based-on-the-viewport/>
  
  And this CSS Tricks article explains the benefits of defining custom properties for the calculation of a proper vh value, which are also used here as an intermediary step towards the desired end result:  
  <https://css-tricks.com/the-trick-to-viewport-units-on-mobile/>
  