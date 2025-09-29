# Tornis - watch and respond
[![npm version](https://badge.fury.io/js/tornis.svg)](https://badge.fury.io/js/tornis)
![the gzip size of Tornis](https://img.badgesize.io/robb0wen/tornis/master/dist/tornis.js.svg?compression=gzip)
![the Brotli size of Tornis](https://img.badgesize.io/robb0wen/tornis/master/dist/tornis.js.svg?compression=brotli)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://opensource.org/licenses/MIT)

![Watch and respond](https://tornis.robbowen.digital/icon.png)

Taking its name from the forest watchtowers of Latvia, Tornis is a minimal JavaScript library that watches the state of your browser's viewport, allowing you to respond whenever something changes. 

__Think of Tornis as a store for your viewport__.

Tornis currently tracks state for:








* Mouse position
* Mouse cursor velocity
* Viewport size
* Scroll position
* Scroll velocity
* Browser position relative to the screen
* Browser velocity relative to the screen
* Device orientation
* Device pixel ratio

You can subscribe to store updates and combine these values to create all sorts of effects.

## When to use Tornis

First and foremost, __Tornis is not a parallax library__. It can of course be used to create parallax effects, but the library itself is concerned only with __tracking the state of your viewport__.

Whilst it is entirely possible to manually track everything in Tornis's store using standard JavaScript event handlers, it can become cumbersome to use them in combination. On top of that, binding complex event handlers to scroll or resize events, for example, can very quickly harm render performance.

Tornis takes a deferred approach. Rather than bind directly to native events, Tornis throttles them and captures only the bare minimum - the updated values. An optimised render loop with the requestAnimationFrame api updates the store and provides the new state to any subscribed functions. This means that your code only runs when the store has changed, and when the browser is ready to render.

## Installation
Tornis can be installed from source and included as a script tag, or from npm. Npm is the preferred method. Open your project directory in your command line or terminal, then run:

```
npm install tornis
```

## The state object
The viewport state can be accessed at any time using the `getViewportState()` function. But the main focus is on watched functions.

You can subscribe a function to the Tornis store by passing it to the `watchViewport()` function. Tornis automatically runs this function whenever the viewport state changes. When it does, the updated viewport state object (seen below) is passed to the watched function as its first parameter.

``` javascript
{
  scroll: {
    changed: Boolean,
    left: Integer,
    right: Integer,
    top: Integer,
    bottom: Integer,
    velocity: {
      x: Integer,
      y: Integer
    }
  },
  size: {
    changed: Boolean
    x: Integer,
    y: Integer,
    docY: Integer
  },
  mouse: {
    changed: Boolean,
    x: Integer
    y: Integer
    velocity: {
      x: Integer
      y: Integer
    }
  },
  position: {
    changed: Boolean,
    left: Integer,
    right: Integer,
    top: Integer,
    bottom: Integer,
    velocity: {
      x: Integer,
      y: Integer
    }
  },
  orientation: {
    changed: Boolean,
    alpha: Integer,
    beta: Integer,
    gamma: Integer
  },
  devicePixelRatio: {
    changed: Boolean,
    ratio: Number
  }
}
```

If its related properties have changed since the last state update, the nested changed properties will be individually set to true. This means that at any given update, you can test for what has and hasn't changed. You should use this to guard your code and ensure that updates only run when they're needed.

You can see an example of this in the standard usage section.

## A note on device orientation
The `deviceorientation` API is currently in a state of flux, with the intention of full deprecation at some point in the future. However, as of publishing, it is still available. On Chrome, `deviceorientation` is deprecated over http, and you will require https for the events to fire. Similarly, on iOS, the API is now blocked behind a flag in the user's Safari settings. The user must enable 'motion and orientation access' for the events to fire.

Once the API is enabled, Tornis doesn't calculate device orientation values exactly as the device does. Instead it resolves for the user's natural resting position by storing the initial device position on page load. All subsequent values reported for alpha, beta and gamma are relative to this initially calibrated position. The function `recalibrateOrientation()` can be used to reset the default position. This function returns an object containing the previous, and incoming calibrated positions.

## Standard usage
``` javascript
// import the Tornis store functions
import { 
  watchViewport, 
  unwatchViewport, 
  getViewportState,
  recalibrateOrientation
} from 'tornis';

// define a watched function, to be run on each update
const updateValues = ({ size, scroll, mouse, position, orientation, devicePixelRatio }) => {
  if (size.changed) {
    // do something related to size
  }
  
  if (scroll.changed) {
    // do something related to scroll position or velocity
  }

  if (mouse.changed) {
    // do something related to mouse position or velocity
  }

  if (position.changed) {
    // do something related to browser window position or velocity
  }

  if (orientation.changed) {
    // do something related to device orientation
  }
  
  if (devicePixelRatio.changed) {
    // do something related to pixel ratio
  }
};

// bind the watch function
// By default this will run the function as it is added to the watch list
watchViewport(updateValues);

// to bind the watch function without calling it
watchViewport(updateValues, false);

// when you want to stop updating
unwatchViewport(updateValues);

// to get a snapshot of the current viewport state
const state = getViewportState();

// to reset the default device orientation.
const calibrationData = recalibrateOrientation();
```

Any watched function will be automatically run whenever any of the associated properties change.

To see a further example of usage, view the demo site at https://rafiulislam.rf.gd

## Legacy script usage
Copy the file `/dist/tornis.js` to your project and include it with a script tag. If you are supporting legacy browsers, you can use the transpiled ES5 version, `/dist/tornis.es5.js`.

``` html
<script src="path/to/tornis.js"></script>
```

Loading Tornis this way creates a global variable `window.__TORNIS`, through which you can access the watch functions.

``` javascript
__TORNIS.watchViewport(updateValues);

__TORNIS.unwatchViewport(updateValues);

const state = __TORNIS.getViewportState();
```
