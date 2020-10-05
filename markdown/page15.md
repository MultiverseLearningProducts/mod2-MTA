# You are done (nearly)

Your app is now feature complete. However there is some refactoring to do.

## Ajax with jQuery

You can refactor the some what clunky XMLHttpRequest and use jQuery and json in it’s place. Have a look at the [documentation](https://api.jquery.com/jquery.ajax/) and refactor your code.

The AJAX call has a few important parameters that you can set. Look at the AJAX call below.

```javascript
$.ajax({
  url: searchPath,
  cache: false,
  dataType: "xml",
  success: function (data) {
    $(data).find("fruit").each(function () {
      $('#searchResults').append($(this).text())
      $('#searchResults').append("<BR/>")
    })
  }
})
```

The first parameter being set is the url that the AJAX call will be requesting. For security reasons, to prevent cross site scripting, this URL must be within the same domain as the web-page itself.

The next parameter, cache, is optional and indicates whether the call can use a cached copy.

The third parameter, datatype, indicates the expected data type, which could be XML or JavaScript Object Notation (JSON), for example.

The next parameter set in this example is the success property. This parameter takes a function that the results of the AJAX calls should be passed into for the webpage to do some work with. In this example, the results are parsed and added to the DOM so that users can see the results.

The final property is set on our AJAX call, as good practice, is the error property so that any error conditions can be handled gracefully.

The jQuery AJAX toolkit supports not only getting data, but also posting data to the server. The default request type is GET. To change a call to a post, you change the value of the
property to 'POST'.