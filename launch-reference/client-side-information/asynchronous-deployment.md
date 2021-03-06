# Asynchronous Deployment

Performance and non-blocking deployment of the JavaScript libraries required by our products is increasingly important to Adobe Experience Cloud users. Tools like [Google PageSpeed](https://developers.google.com/speed/pagespeed/insights/) recommend that users change they way they deploy the Adobe libraries on their site. This article explains how to use the Adobe JavaScript libraries in an asynchronous fashion.

## Synchronous vs asynchronous

### Synchronous deployment

Often, libraries are loaded synchronously in the `<head>` tag of a page. For example:

```markup
<script src="example.js"></script>
```

By default, the browser parses the document and reaches this line, then starts to fetch the JavaScript file from the server. The browser waits until the file is returned, then it parses and executes the JavaScript file. Finally, it continues parsing the rest of the HTML document.

If the parser comes across the `<script>` tag before rendering visible content, content display is delayed. If the JavaScript file being loaded is not absolutely necessary to show content to your users, you are unnecessarily requiring your visitors to wait for content. The larger the library, the longer the delay.  For this reason, website performance benchmark tools like Google PageSpeed or Lighthouse often flag synchronously loaded scripts.

Tag Management libraries can quickly grow large if you have a lot of tags to manage.

### Asynchronous deployment

You can load any library asynchronously by adding an `async` attribute to the `<script>` tag.  For example:

```markup
<script src="example.js" async></script>
```

This indicates to the browser that when this script tag is parsed, the browser should begin loading the JavaScript file, but instead of waiting for the library to be loaded and executed, it should continue to parse and render the rest of the document.

## Considerations to asynchronous deployment

If you choose to load Launch asynchronously, there are a few things to consider.

### Timing

As described above, in synchronous deployments, page rendering pauses while the Launch library is loaded and executed. This means that events that happen after the library is loaded \(Page Bottom, DOM Ready, Window Loaded, etc\) always reliably happen after the `_satellite` object is available.

In asynchronous deployments, the page rendering does not pause for the library to be loaded.  This means that the browser sequence of page load events may no longer occur where you expect them to - in relation to your library loading.  Some examples:

1. A rule that uses `Core - Library Loaded` as an event may be triggered before your data layer is fully loaded.  This may result in Rule Actions executing with missing data because the data was not yet on the page.
2. A rule that uses `Core - DOM Ready` as an event may be triggered before your library has been fully loaded.  In this case, the rule actions will be delayed until the Library is fully loaded.  Launch will make sure that rules still execute in the logical order, but it may be later than you expected.

These kinds of problems can be mitigated by making tweaks to your rule configuration.  As an example, instead of having a rule triggered by `Core - Library Loaded`, you could instead use a Direct Call rule that is called as soon as your data layer finishes loading.

If you see things occurring out of order - or occurring in different order inconsistently - it is likely that you have some timing issues to work through.

Deployments that require precise timing may need to make more use of eventHandlers and direct call rules in order to make their implementations more robust and consistent.

### Page Bottom event type

Another consideration is that Launch has always provided a Page Bottom event type that allows users to fire a rule at the precise moment the bottom of the body tag is reached by the browser parser. Because the Launch runtime will likely finish loading after the page bottom has been reached, the Page Bottom event type may not fire associated rules at the time you may expect. For this reason, when loading Launch asynchronously, you should not use the Page Bottom event type. Instead, consider the Library Loaded, DOM Ready, Window Loaded, or other event types.

## Loading the Launch embed code asynchronously

Launch provides a toggle to turn on asynchronous loading when creating an embed code when you configure an [environment](../publishing/environments.md). You can also configure asynchronous loading yourself:

1. Add an async attribute to the `<script>` tag to load the script asynchronously.

   For the Launch embed code, that means changing this:

   ```markup
   <script src="//www.yoururl.com/launch-EN1a3807879cfd4acdc492427deca6c74e.min.js"></script>
   ```

   to this:

   ```markup
   <script src="//www.yoururl.com/launch-EN1a3807879cfd4acdc492427deca6c74e.min.js" async></script>
   ```

2. Remove any code you may have previously added at the bottom of your tag:

   ```markup
   <script type="text/javascript">_satellite.pageBottom();</script>
   ```

   This code tells Launch that the browser parser has reached the bottom of the page. Since Launch likely will not have loaded and executed before this time, calling `_satellite.pageBottom()` results in an error and the Page Bottom event type may not behave as expected.

