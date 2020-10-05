# WebAudio API and Inheritance

*   _1.5 Establish the scope of objects and variables Define the lifetime of variables; keep objects out of the global namespace; use the "this" keyword to reference an object that fired an event; scope variables locally and globally_

We have frequencies. Now it is time to use them. Each instance of a Note has its own frequency. We can use the browser's built in Web Audio API to play those frequencies.

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vS1_P6OsA3IvG1e846cusmYDqydZ7GTiI0uMwAWsP6RPwFoX8V3y45jzOsa4JvG6APcn0NJnZKkXq4N/embed?start=false&loop=true&delayms=3000" frameborder="0" width="100%" height="444" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

One important thing to know about the Web Audio API is that before initialising the `AudioContext` and playing a frequency, there must be an interaction by the user, either a gesture or a click event. This is to prevent the Web Audio API starting on page load and starting to emit sound that the user has no control over.

We also want only 1 instance of the `AudioContext` on our page but many instances of oscillators at different frequencies. Lets solve this using inheritance.

## Playing notes

Here is how to play a simple note (you can paste this into your browsers console to hear it).
```javascript
const context = new AudioContext()
const o = context.createOscillator()
o.frequency.value = 4400
o.connect(context.destination)
o.start()
o.stop(context.currentTime + 1)
```
You can think of a note as an object. We’ll want to create lots of notes in our programme, they will all be similar, but might have different frequencies and start at different times. This is a good candidate for using classes.

Each note will rely on the audio context. We only need <u>one</u> of these, all the notes we create will originate from and be connected to this context. The frequency of an individual note only need to exist in the scope of an instance of a note.

What we need is a singular context object, and then note objects that inherit access to that audio context. We want to end up making new notes like this:
```javascript
new Note(4400)
```
The Note class needs to have access to the audio context. But we also want to have the audio context kept out of global scope. It is best practice to keep variables out of global scope. In global scope variables can be read by any part of your program, and variables can be reassigned by any code in the program.

Another issue is you can only create a new AudioContext after a user has interacted with the page. You’ll see errors in the console if you try to do this. Because of this lets define our base class MyAppAudio like this:
```javascript
class MyAppAudio {
  static context = undefined;
  static setContext = () => {
    this.context = new AudioContext()
  }
}
```
The static keyword creates a class property and a class function. It means we can access them without creating an instance with the **new** keyword. This keeps the AudioContext out of global scope and contains it in a class. We can then extend this class with our Note class and then access the context via inheritance.
```javascript
class Note extends MyAppAudio {
  constructor(freq) {
    super()
    if (!this.constructor.context) this.constructor.setContext()
    const o = this.constructor.context.createOscillator()
    o.frequency.value = freq
    o.connect(this.constructor.context.destination)
    o.start()
    o.stop(this.constructor.context.currentTime + 1)
  }
}
```
Notice the call `super()`? We need that in out constructor when we are inheriting from classes. Can you also spot how we reference the inherited `context` property? Then we need a little guard clause before we create a note to initialise the `new AudioContext` if it is the first interaction.

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vTpt_GcwPW3lISEoKCt0fDUykw0MShTJH-dbL7BI_4NcM6nsuS0YzgVdbtK-gE6PSK5-1OODOgZJY75/embed?start=false&amp;loop=true&amp;delayms=3000" frameborder="0" width="100%" height="444" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

in the example above we used the **class** keyword. This is just syntactic sugar for the javascript prototypal inheritance that is going on behind the scenes. Have a look through the example below that does the same thing only this time using the prototype property.
```javascript
function AppAudio () {}
AppAudio.prototype.context = undefined
AppAudio.prototype.setContext = function() {
	this.context = new AudioContext()
}

const Note = function (freq) {
	if (!this.constructor.context) this.constructor.setContext()
	const o = this.constructor.context.createOscillator()
	o.frequency.value = freq
	o.connect(this.constructor.context.destination)
	o.start()
	o.stop(this.constructor.context.currentTime + 1)
}
Note.prototype = Object.create(new AppAudio())

function play (freq) {
	new Note(freq)
}
```
Considering the above can you update your Note class to inherit from a Base class called `MyAppAudio`?

![UML diagram](https://user-images.githubusercontent.com/4499581/87775791-930e8d80-c81e-11ea-8b28-52faf685311d.png)