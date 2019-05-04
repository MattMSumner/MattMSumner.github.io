---
layout: post
title: JavaScript Audio API
date: 2016-06-08 10:24:13
categories: es6 javascript web
excerpt: How to draw waveforms using JavaScript.
---

*This article was originally posted on the [thoughtbot blog
here](https://thoughtbot.com/blog/javascript-audio-api)*

I recently had the opportunity to play around with JavaScript's [Web Audio API].
There were a couple of features we ended up leveraging, and some I'd like to
explore in the future (check out some of the audio nodes you can apply;
[ConvolverNode](https://developer.mozilla.org/en-US/docs/Web/API/ConvolverNode),
[DynamicsCompressorNode](https://developer.mozilla.org/en-US/docs/Web/API/DynamicsCompressorNode)
or
[PannerNode](https://developer.mozilla.org/en-US/docs/Web/API/PannerNode)!).
But today I wanted to talk about [waveforms].

[web audio api]: https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API
[waveforms]: https://en.wikipedia.org/wiki/Waveform

Waveforms are a visual representation of a sound wave. In terms of audio,
there are several types of waveforms depending on what your axes represent.
The x-axis will show the time and the y-axis will show the min/max values from a
sample group of the audio [pulse code modulation] (PCM).

[pulse code modulation]: http://wiki.multimedia.cx/?title=PCM

## Fetching audio

The first thing we'll need to do is fetch our audio from the server. I've made an
ES6 class to handle fetching the audio file and setting up some of the
JavaScript objects we'll need for interacting with our audio.

```js
// app/assets/javascripts/audio.es6

class Audio {
  constructor({src}) {
    this.context = this._createAudioContext();
    this._loadTrack(src);
  }

  _createAudioContext() {
    if (!window.audioContextInstance) {
      window.AudioContext = window.AudioContext || window.webkitAudioContext;

      if (window.AudioContext) {
        window.audioContextInstance = new AudioContext();
      } else {
        alert('Web Audio API is not supported in this browser');
      }
    }

    return window.audioContextInstance;
  }

  _loadTrack(url) {
    const request = new XMLHttpRequest();
    request.open('GET', url, true);
    request.responseType = "arraybuffer";

    request.onload = () => {
      this.context.decodeAudioData(request.response, buffer => {
        this.buffer = buffer;
      }, function() {});
    }
    request.send();
  }
}
```

This takes the `src` for our audio file and sets up an audio context and then
loads the file. Once the file is loaded, it sets up an audio buffer with the
loaded track. Now that we have our track loaded into a buffer, we can start
interacting with it. We can invoke this class with
`var audio = new Audio({src: "some-audio-url"});`.

## Drawing a waveform

Next, we'll want to sample our audio buffer to extract the min/max values from
our PCM. Once we have that, we can create a function to display our waveform. To
achieve this I've created a Waveform class:

```js
// app/assets/javascripts/waveform.es6

const PEAK_COUNT = 6000;
class Waveform {
  constructor({svgPathId, buffer}) {
    this.svgId = svgId;
    this.buffer = buffer;
    this.draw();
  }

  draw() {
    getElementById(this.svgPathId).setAttribute('d', this.svgPath());
  }

  svgPath() {
    const peaks = this._getPeaks(PEAK_COUNT);
    const totalPeaks = peaks.length;

    let d = '';
    for(let peakNumber = 0; peakNumber < totalPeaks; peakNumber++) {
      if (peakNumber%2 === 0) {
        d += ` M${~~(peakNumber/2)}, ${peaks.shift()}`;
      } else {
        d += ` L${~~(peakNumber/2)}, ${peaks.shift()}`;
      }
    }
    return d;
  }

// ...
}
```

We've used the [bitwise not] operator `~` here to ensure that some values are
whole numbers.

[bitwise not]: http://james.padolsey.com/cool-stuff/double-bitwise-not/

I've omitted the `_getPeaks` function. All this function does is take the
highest and lowest values over a range within the audio channel's data. It
returns an array of sampled values.

The SVG path here is using `M` and `L` followed by a vector to draw our
waveform. The `M` stands for move and the `L` means draw a line from the current
location along the vector.

This takes an ID for an SVG path element and an audio buffer. We define the
sample rate as the constant `PEAK_COUNT`; this is how many sections we'll split
our audio buffer into. The `svgPath` function converts our min/max sampling into
a SVG path by drawing a vertical line for each min/max pair. The `_getPeaks`
function is what actually performs the sampling. To draw this, we
need an SVG element on the page and to pass an `id` and our audio buffer:

```html
<svg viewBox="0 -1 6000 2" preserveAspectRatio="none">
  <g>
    <path id="waveform" d=""/>
  </g>
</svg>

<script>
  new Waveform({svgPathId: "#waveform", buffer: audio.buffer});
</script>
```

The SVG's viewBox defines how much of the coordinate system we would like to
show in our element.
The samples returned from `_getPeaks` will be between -1 and 1.
Here I've set the viewBox to match our sampling rate for the x-axis and to go
from -1 to 1 on the y-axis.
This means that the waveform will stretch to the
width and height of our SVG.
Along with setting `preserveAspectRatio`, our
waveform will now scale to fill the whole SVG. One of the cool things about
using an SVG here is how easy it is to style using CSS.

## Demo

I threw all of this into a [small Ember app] to show it in action (the repo can
be found [here]).

[small Ember app]: https://appallingfarrago.com/waveforms
[here]: https://github.com/MattMSumner/waveforms
