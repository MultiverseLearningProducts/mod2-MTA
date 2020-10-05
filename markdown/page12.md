# Player

We are going to make a player to play and visualise our tracks. For times when we want to render graphics or draw to the DOM there is the `<canvas>` element. This is going to be at the heart of our `Player` component.

|Description|UML|
|:---|:---|
|The `Player` class should have an instance of the `Worker` like the `Grid` does, because the `Player` will need to play a track so we will want a `_tick()` function to consume and play the bars of the track.<br/><br/>We want somewhere to store the track that the player is going to play. Then add functions to `load(track)`, `play()`, & `stop()` a track.<br/><br/>The `_render()` function should return the HTML that will make up our player interface.|![UML Player class](https://user-images.githubusercontent.com/4499581/89511951-71277b80-d7ca-11ea-9abb-cb58c683548c.png)|

## `atob()`

Previously we constructed our tracks so when they were are clicked they will call:

```javascript
onclick="player.load('${btoa(JSON.stringify(track))}')"
```

To decode the string that `load(track)` will receive we need to call the decode version of `btoa()` which is `atob()` that will take a base64 encoded string and return you a JSON stringified track. You can then hide the grid, show the player, add the `JSON.parse` version of the track into the `Player`'s track property and then call `_render()`.

# 👩🏾‍💻🧑🏽‍💻👨🏻‍💻

Create a `Player` class using the UML diagram above as a guide. When a user clicks on a track it should hide the grid and display the player. The player should have the track that was clicked loaded into the `track` property.

## `play()`

The track is in the `Player` can you play it? Use the worker to get a 'tick'. Then play the bars in the loaded track.

# Visualising the Track

We will be able to hook into the audio data by inserting an "Analyser" node into our audio graph. That will give us a stream of audio data that we can use to draw on the canvas element.

Before we look at that let first of all try to draw some simple lines and shapes on the canvas.

<iframe src="https://docs.google.com/presentation/d/e/2PACX-1vS2qKAx1X2_HWsPd0AdE5QN-jF0PgJGhjA_1n-rp09m34Zo8t4EE8j9JE2XjI2CGO-RVVSRsGjAL34w/embed?start=false&loop=false&delayms=3000" frameborder="0" width="100%" height="444" allowfullscreen="true" mozallowfullscreen="true" webkitallowfullscreen="true"></iframe>

# 👩🏾‍💻🧑🏽‍💻👨🏻‍💻

```javascript
_render() {
    const playerElement = `
        <article style="position:relative;">
            <h2>${this.track.trackname}</h2>
            <canvas></canvas>
            <nav style="position:absolute:bottom:0;">
                <button onclick="player.play()">Play</button>
                <button onclick="player.stop()">Stop</button>
            </nav>
        </article>
    `
    $("#player").append(playerElement)
    $('canvas').get(0).setAttribute('width', $("#player").width())
    $('canvas').get(0).setAttribute('height', $("#player").height() - 130)
    this.canvas = {
        ctx: $('canvas').get(0).getContext('2d'),
        w: $('canvas').width(),
        h: $('canvas').height()
    }
    this.canvas.ctx.fillStyle = "burlywood"
}
```

Can you spot the canvas context interface? We call `getContext('2d')` to access an instance of the canvas context. It is calling commands on this interface that enable us to draw on our canvas. Given the `_render()` function above. Can you draw a series of bars on your canvas?
