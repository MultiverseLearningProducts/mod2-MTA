# Tracks

|||
|:--|:--|
Create a `Tracks` class by following the UML diagram on the right.<br/><br/>Assign a web socket connection to `ws` property. `tracks` is going to be your local state of tracks received via the web socket. Remember to register an `onmessage` handler in the constructor function that listens for "collect" or an updated list of tracks. `save(track)` should take a track instance and save it to `localStorage`. `_send()` can be called on "collect" to send all the tracks from your `localStorage`, and when a new track is saved you can call `_send()` so everyone sees your new track. `_render()` this should empty your parent HTML element that will hold the list of tracks, and then create the HTML for each track (including an onclick event to load the track into a `Player`) then add it into the DOM|![UML Track class](https://user-images.githubusercontent.com/4499581/87775782-90139d00-c81e-11ea-93e2-169f23427e9b.png)

![example of tracks rendered in the DOM](https://user-images.githubusercontent.com/4499581/89508534-f3617100-d7c5-11ea-9301-316ed45acd3b.png)

Above is an example of the tracks rendered in an `<aside>` element next to the composing grid interface.

## onclick

In the next section we are going to build out the `Player` component. In preparation for this we should have our track call the following function when it is clicked:

```html
<button onclick="player.load('${btoa(JSON.stringify(track))}')">${track.trackname}
</button>
```

Earlier we looked at serialising data. Here we are using a a function called `btoa()` which is part of the browser's javascript API. It will base64 encode our track. We have to do this as just stringifying our track and including it inline as an argument to a javascript function, is not enough to escape all the quotes and other punctuation.

We are going to turn this:

{"trackname":"rain down","data":[[1047,523.3,261.6,130.8],[1175,587.3,293.7,146.8],[1319,659.3,329.6,164.8],[1397,698.5,349.2,174.6],[1480,740,370,185],[1568,784,392,196],[1760,880,440,220],[1976,987.8,493.9,246.9],[1047,523.3,261.6,130.8],[1175,587.3,293.7,146.8],[1319,659.3,329.6,164.8],[1397,698.5,349.2,174.6],[1480,740,370,185],[1568,784,392,196],[1760,880,440,220],[1976,987.8,493.9,246.9],[1047,523.3,261.6,130.8],[1175,587.3,293.7,146.8],[1319,659.3,329.6,164.8],[1397,698.5,349.2,174.6],[1480,740,370,185],[1568,784,392,196],[1760,880,440,220],[1976,987.8,493.9,246.9],[1047,523.3,261.6,130.8],[1175,587.3,293.7,146.8],[1319,659.3,329.6,164.8],[1397,698.5,349.2,174.6],[1480,740,370,185],[1568,784,392,196],[1760,880,440,220],[1976,987.8,493.9,246.9],[1047,523.3,261.6,130.8],[1175,587.3,293.7,146.8],[1319,659.3,329.6,164.8]],"city":"","countryCode":""}


Into this:

<div style="word-wrap:break-word;max-width:100%;">eyJ0cmFja25hbWUiOiJyYWluIGRvd24iLCJkYXRhIjpbWzEwNDcsNTIzLjMsMjYxLjYsMTMwLjhdLFsxMTc1LDU4Ny4zLDI5My43LDE0Ni44XSxbMTMxOSw2NTkuMywzMjkuNiwxNjQuOF0sWzEzOTcsNjk4LjUsMzQ5LjIsMTc0LjZdLFsxNDgwLDc0MCwzNzAsMTg1XSxbMTU2OCw3ODQsMzkyLDE5Nl0sWzE3NjAsODgwLDQ0MCwyMjBdLFsxOTc2LDk4Ny44LDQ5My45LDI0Ni45XSxbMTA0Nyw1MjMuMywyNjEuNiwxMzAuOF0sWzExNzUsNTg3LjMsMjkzLjcsMTQ2LjhdLFsxMzE5LDY1OS4zLDMyOS42LDE2NC44XSxbMTM5Nyw2OTguNSwzNDkuMiwxNzQuNl0sWzE0ODAsNzQwLDM3MCwxODVdLFsxNTY4LDc4NCwzOTIsMTk2XSxbMTc2MCw4ODAsNDQwLDIyMF0sWzE5NzYsOTg3LjgsNDkzLjksMjQ2LjldLFsxMDQ3LDUyMy4zLDI2MS42LDEzMC44XSxbMTE3NSw1ODcuMywyOTMuNywxNDYuOF0sWzEzMTksNjU5LjMsMzI5LjYsMTY0LjhdLFsxMzk3LDY5OC41LDM0OS4yLDE3NC42XSxbMTQ4MCw3NDAsMzcwLDE4NV0sWzE1NjgsNzg0LDM5MiwxOTZdLFsxNzYwLDg4MCw0NDAsMjIwXSxbMTk3Niw5ODcuOCw0OTMuOSwyNDYuOV0sWzEwNDcsNTIzLjMsMjYxLjYsMTMwLjhdLFsxMTc1LDU4Ny4zLDI5My43LDE0Ni44XSxbMTMxOSw2NTkuMywzMjkuNiwxNjQuOF0sWzEzOTcsNjk4LjUsMzQ5LjIsMTc0LjZdLFsxNDgwLDc0MCwzNzAsMTg1XSxbMTU2OCw3ODQsMzkyLDE5Nl0sWzE3NjAsODgwLDQ0MCwyMjBdLFsxOTc2LDk4Ny44LDQ5My45LDI0Ni45XSxbMTA0Nyw1MjMuMywyNjEuNiwxMzAuOF0sWzExNzUsNTg3LjMsMjkzLjcsMTQ2LjhdLFsxMzE5LDY1OS4zLDMyOS42LDE2NC44XV0sImNpdHkiOiIiLCJjb3VudHJ5Q29kZSI6IiJ9</div>

Our `load(track)` function in our `Player` will be passed this encoded string which it will then decode and load into the player.

