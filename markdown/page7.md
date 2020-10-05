# WebWorkers

For our audio app to work we need something in our programme that acts like a beating heart or a ticking clock. This tick will move the playhead from bar to bar and enable us to trigger the playing of notes from our audio grid.

As you can imagine it will be important that our beat is even and steady. We don’t want the regular ticking to be screwed by other demands on the javascript execution thread that might be made, by animations for example. This is a good case for a WebWorker

## What is a WebWorker?

*   _2.5 Create a web worker process Start and stop a web worker; pass data to a web worker; configure timeouts and intervals on the web worker; register an event listener for the web worker; limitations of a web worker_

A WebWorker is a separate process that we can run in the browser.

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vSFOdHco9hALnllqPomOzNP9naeYSLoOv8K0GrNMIuA-DVjNLbQ5eOJ9i9H-ARbl0n3XyBEKlqGDjmc/embed?start=false&amp;loop=false&amp;delayms=3000" frameborder="0" width="100%" height="444" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

## Setting up the audio engine’s tick

*   Create a WebWorker when the application loads in the browser
*   Wire up a start message that is sent when the play button is pressed
*   Upon receiving the start message your WebWorker should send messages on a steady beat
*   When the stop button is pressed you should stop the regular “tick” messages coming from the WebWorker
*   You should also terminate the WebWorker when the page unloads

## Knowledge test

???
[q] Which method do you call on an instance of a WebWorker to pass a message from the main thread to the worker thread?
[ ] worker.send(msg)
[ ] worker.post(msg)
[#] worker.postMessage(msg)
???

???
[q] Which one of these is NOT a limitation of a WebWorker?
[ ] You cannot load images
[ ] You cannot access the DOM
[#] You cannot connect to a WebSocket
???

## Moving the bar

*   _1.3 Apply styling to HTML elements programmatically Change the location of an element; apply a transform; show and hide elements_

Use this beating heart to move the bar across the grid. Something like this.

![bar looping](https://user-images.githubusercontent.com/4499581/71742206-e1e36f80-2e58-11ea-84da-70b028c93d5a.gif)

We have a Note class in our app. All these notes that we have really need organising. We want to be able to "play" whats on the grid and "record" our track. There is enough functionality needed for us to create a Grid class that will organise and keep track of the bars, and beats.

```javascript
class Grid {
	constructor() {
	  this.worker = new Worker("/worker.js")
	  this.bar = 0
	  this.grid = [
	    [1047.0, 1175, 1319, 1397, 1480, 1568, 1760, 1976],
	    [523.3, 587.3, 659.3, 698.5, 740, 784, 880, 987.8],
	    [261.6, 293.7, 329.6, 349.2, 370, 392, 440, 493.9],
	    [130.8, 146.8, 164.8, 174.6, 185, 196, 220, 246.9]
		].map(row => row.map(freq => new Note(freq)))
		this.worker.onmessage = this._tick.bind(this)
	}
	play() {
	  this.worker.postMessage("start")
	}
	stop() {
	  this.worker.postMessage("stop")
	  this.grid.forEach(row => row.forEach(note => {
		note.selected = false
		note.renderSelected()
	  }))
	  this.bar = 0
	}
	_tick() {
	  this._play()
	  this.bar = this.bar === 7 ? 0 : this.bar + 1
	}
}
```
Now we have some updates to make to our Notes class (but we will come back to that). Lets go through this class. In the constructor function we set up some state, the worker, the bar and the grid.

Can you see how we set up the web worker? The grid starts off as a multi-dimensional array. Then we map over it returning an array of arrays of notes set to a particular frequency.

Finally we set an event listener on the worker that will call `this._tick` when it receives messages. Notice we are doing the same thing as we did earlier, using bind to bind the context of the class to `this._tick`.

`this.play` sends a "start" message to the web worker, `this.stop` sends a "stop" message to the worker, but also re-sets the bar and grid. `this._tick` is called whenever we receive a message from the worker, and it increments the bar, and calls a function that we have not created yet called `this._play`. We will come back to that.

### Worker

The web worker just sends messages at regular intervals.
```javascript
let interval;
onmessage = function (msg) {
    if(msg.data === "start") {
        interval = setInterval(() => {
            postMessage("tick")
        }, 300)
    }
    if(msg.data === "stop") {
        clearInterval(interval)
    }
}
```
In this file the meaning of `this` is the web worker itself. So when we reference `onmessage` and `postMessage` javascript looks in the local scope for definitions of these methods. They are not in the local scope, so javascript goes up one level to the parent scope (the worker) and finds that these methods are defined in that scope. `clearInterval` is defined in the global scope (the window object).

We could also write the same method like this:
```javascript
let interval;
this.onmessage = function (msg) {
    if(msg.data === "start") {
        interval = setInterval(() => {
            this.postMessage("tick")
        }, 300)
    }
    if(msg.data === "stop") {
        clearInterval(interval)
    }
}
```

Now we have a beat. Can you console log out the bar progressing from 0 - 7?

```
const grid = new Grid()
```
Add the call to `new Grid()` to your javascript file and see if you can start and stop a regular beat coming into your app from the web worker. You will have to wire up your control buttons to call `grid.start` on click:

```html
<button onclick="grid.start()">Start</button>
```

### Note UI

We need to update our `Note` class to have a method we can call that will temporary give the note a highlight when the bar is being played. Add the following to your `Note` class:

```javascript
highlight() {
    this.el.addClass("highlight")
    setTimeout(() => {
        this.el.removeClass("highlight")
    }, 300)
    return this
}
```

### `Grid._play()`

We need to create that `_play()` method mentioned earlier and add it to the `Grid` class.

```javascript
_play() {
    this.grid
        .map(row => row[this.bar])
        .flat()
        .map(note => note.highlight())
}
```

In the `_play` method above we map over the rows that form the grid and return only the note that corresponds to the current bar. That leaves us with a structure that would look something like this:

```javascript
[
  [Note],
  [Note],
  [Note],
  [Note]
]
```

Calling `.flat()` on this structure will remove the nesting and flatten the array to this:

```javascript
[Note, Note, Note, Note]
```
Next we map over each Note instance and call `note.highlight()` on each one. Why do you think it is important to return `this` from the `.highlight` method?
