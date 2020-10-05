# Sharing tracks over WebSockets

*   _2.4 Implement a callback Receive messages from the HTML5 WebSocket API_

You have tracks saved in localStorage. Let's make them available so everyone else can enjoy them. We are going to use WebSockets to connect to a relay server that we can send our tracks to and have them broadcasted to everyone connected.

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vRcUuangOTH4PvIyU9V409kMkt2SOKOlJ3QZ1ywh8MO8ZJceAG7xohX7H5L7k41tHSpqPTmpoig8UrP/embed?start=false&amp;loop=false&amp;delayms=3000" frameborder="0" width="100%" height="444" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

You should connect to this web socket endpoint when your app loads:
```javascript
    const ws = new WebSocket('ws://fathomless-reaches-81353.herokuapp.com/socket')
```
You can visit [http://fathomless-reaches-81353.herokuapp.com/](http://fathomless-reaches-81353.herokuapp.com/) for instructions about how to use this relay service. It basically works something like this.

![illustration of the web socket relay server](https://user-images.githubusercontent.com/4499581/71779386-1fa5dc80-2fae-11ea-8c0c-aa179892c729.png)

Once you have a websocket instance you can add event listeners to it like this:
```javascript
    ws.onmessage = function (msg) {
        msg.data // payload from the server
    }
    ws.onclose = function () {
        // do something when the socket connection closes
    }
```
Each peer stores their tracks in localStorage.

```javascript
_save(track) {
    let tracks = JSON.parse(localStorage.getItem("tracks") || "[]")
    localStorage.setItem("tracks", JSON.stringify([...tracks, track]))
}
```
When you save tracks you have to read them out of localStorage, deserialise them, then append your new track. Then do the reverse and serialise the tracks and re-writing the updated data in localStorage.

Upon connection peers receive a `msg.data` that equals “collect”, you should then send your tracks from localStorage to the socket server. The socket server then broadcasts those tracks to all the connected peers (including you). Anything that is not a “collect” msg is going to be an array of serialised tracks.

No tracks are stored on a central server, the socket server is “stateless”, tracks are distributed among the connected peers. Update your code to render tracks from the incoming messages from your WebSocket NOT from localStorage!

You can use the spread operator to append items to an array. It’s similar to the way the spread operator works with objects:
```javascript
    const a = [1,2,3]
    const b = [4,5,6]
    const c = [...a, ...b]
    c // [1,2,3,4,5,6]
```
There is enough going on here to warrant an object in our programme that deals with tracks. Let us add a `Tracks` class that will encapsulate the logic and state of all our tracks.