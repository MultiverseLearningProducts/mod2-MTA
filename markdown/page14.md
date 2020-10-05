# Geolocation

Our Audio app is so amazing we need to think about it being used in lots of different countries. When users create tracks it would be so cool to know which country they created the track in.

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTjDs3Rv-Kup2MNhAGWb564oSe1P4n8jRFtC6j3hpaBHMCHShNsTz67pLgBzjkKHihd8ngnKGCsKj6M/embed?start=false&loop=false&delayms=3000" frameborder="0" width="100%" height="444" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

```javascript
if ("geolocation" in navigator) {
    navigator.geolocation.getCurrentPosition(position => {
        const {latitude, longitude} = position.coords
        console.log({latitude, longitude})
    }
}
```

Useful, but latitude and longitude don't mean much to us humans. What we'll have to do is turn these coords into a human readable place name.

For this we can turn to an API. Our Audio app can make a call out to the internet with those coords and get back in return a data object that will tell us which country the track was created in.

## XMLHttpRequest

The browser has this capability built into it. You'll find on the `window` object a constructor named `XMLHttpRequest`. Can you find it? Lets go through the steps to form an http request using this API.

1. Create an instance of a request
1. Add the function that will handle the response
1. Open the URL
1. Initiate the request

```javascript
const request = new XMLHttpRequest()
```

Now you can write the function to handle the response. What is somewhat unusual about XMLHttpRequest is the response data is accessible through the `this` context of the response handler.

```javascript
const onGeoRequest = function () {
    const [addressparts] = this.responseXML.getElementsByTagName('addressparts')
    const countryCode = addressparts.getElementsByTagName('country_code')[0].innerHTML
    console.log('countryCode', countryCode)
}
```

Can you see us accessing our data through `this` - the function does not receive an argument. Let us add our handler to the request.

```javascript
request.addEventListener('load', onGeoRequest)
```

Then we keep adding to the `request` instance we created by adding the URL we want to open. Then actually executing it.

```javascript
request.open("GET",'https://eu1.locationiq.com/v1/reverse.php?key=YOUR_API_KEY&lat=YOUR_LATITUDE&lon=YOUR_LONGITUDE&format=xml')
request.send()
```

Now our request URL needs some explanation. We are using an on-line service [Locationiq](https://eu1.locationiq.com/) to reverse Geo-lookup. That means we start with lat and lng and end with a place name. The reverse of starting with a place name and ending up with a lat and lng.

You'll need to register and get yourself a free API key to make requests. The url is formed with 4 query parameters:

1. key="pk.22b24860e80eaf7b4fe6e1c481d89b97"
1. lat=51.4850816
1. lon=-0.1015808
1. format=xml

These are separated by the "&" character. Can you put all this together so you end up with a `countryCode` for example **GB** or **FR**.

With this country code you can have a little fun displaying a flag next to the track. Check out [Country Flags API](https://www.countryflags.io/) and get yourself a little flag like this:

```html
<img src="https://www.countryflags.io/:country_code/:style/:size.png">
<img src="https://www.countryflags.io/fr/flat/32.png">
```
Voilà ![French Flag (https://www.countryflags.io/fr/flat/32.png)