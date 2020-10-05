# Canvas & Request Animation Frame

![track being visualised](https://user-images.githubusercontent.com/4499581/90388072-2be33380-e07f-11ea-98f5-1001f1c5182a.gif)

Above is the kind of thing we are aiming for. We need to extend 2 of our classes. We are going to add an analyser node to our audio graph when we play tracks using the MyAppAudio base class. We also need to add some code to animate the bars in the Player class.

## Extend the MyAppAudio class

|instructions|UML|
|:-----------|:--|
|Can you extend this class by adding two new static properties; analyser and audioData. All of these will be set when you call `MyAppAudio.setContext()` as the analyser is created from the AudioContext instance.|![MyAppAudio UML diagram](https://user-images.githubusercontent.com/4499581/90389291-2f77ba00-e081-11ea-8e0a-7b912a62eb38.png)|

Adding an analyser node means when we connect our source (an oscillator) through this node, we will be able to access some of the audio data. That snapshot of data we are going to place in an Uint8Array - thats an array with a fixed size, which we can iterate over, and use those values to set the height of our bars as the audio plays.

```javascript
class MyAppAudio {
    static context = undefined
    static analyser = undefined
    static audioData = undefined
    static setContext = () => {
        this.context = new AudioContext()
        this.analyser = this.context.createAnalyser()
        this.audioData = new Uint8Array(this.analyser.frequencyBinCount)
    }
}
```

Instances of the Note class need to know if they should pass through the analyser node or not. Can you think of a way to refactor the `Note.play()` method to conditionally pass through the analyser node if it is being played back by the Player?

## Extend the Player class

How too animate? Think about a flick book.
![flick book](https://images.squarespace-cdn.com/content/v1/5b1e8e5e12b13fe583922460/1528835374841-5QV6TRZ76LEK6GAU8DQQ/ke17ZwdGBToddI8pDm48kDwcMOkWjB-dh5UJuaCAkzJZw-zPPgdn4jUwVcJE1ZvWQUxwkmyExglNqGp0IvTJZUJFbgE-7XRK3dMEBRBhUpx7gYYqsE3-d3T-rAujCsKB6Ebg7BE-3tI4Wg3H92DuXshSGXJoxzxQyr_LXYiHo9c/Meteor-anim-gif+2.gif?format=1500w)
To get the impression of movement you need to flick the pages pretty quickly. We need to do something similar. We can wipe our canvas clean and draw on it. If we do this fast enough, it will look like things are moving. How about 60 times a second?

The browser API has a function we can call called `requestAnimationFrame`. If we call `requestAnimationFrame(this._draw.bind(this))` with a function, it will cause that function to be called with every paint of the browser (about 60 times a second). If you don't call the function the animating will stop. We will have to use `.bind` to preserve our executing context. Below is a suggested `_draw()` function.

```javascript
_draw() {
    if (!MyAppAudio.context) MyAppAudio.setContext()
    if (this.isPlaying) requestAnimationFrame(this._draw.bind(this))
    MyAppAudio.analyser.getByteFrequencyData(MyAppAudio.audioData)
    this.canvas.ctx.clearRect(0, 0, this.canvas.w, this.canvas.h)
    const barWidth = 32
    
    MyAppAudio.audioData.reduce((x, barHeight) => {
        const y = barHeight / 2
        this.canvas.ctx.fillRect(x, this.canvas.h - y, barWidth, y)
        return x + barWidth + 1
    }, 0)
}
```
Can you read this function? 

???
[q] what variable is being used to stop the animation?
[ ] (!MyAppAudio.context)
[#] (this.isPlaying)
[ ] this.canvas.ctx.clearRect 
???

???
[q] Which array function is used to iterate over the data?
[#] reduce
[ ] forEach
[ ] map
???

`MyAppAudio.analyser.getByteFrequencyData(MyAppAudio.audioData)` this fills the `MyAppAudio.audioData` array with values. We then wipe the canvas clean and then draw a series of bars across the canvas. How does the `x` value increase? What happens when you alter the `barWidth` variable?