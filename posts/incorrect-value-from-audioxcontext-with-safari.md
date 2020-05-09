---
title: HTML5 AudioContext doesn’t update correct values when switch page to background in Safari
date: 2020-05-09
tags: ["Safari", "AudioContext"]
---

Last week Cedric and I made a new product named
“[Mornin](https://mornin.fm "an instant audio conferencing service")”,
an instant audio conferencing service.

Mornin uses HTML5 AudioContext to handle the media stream and get data from analyzer to draw frequency bars.
But I found a strange problem with Safari:
when the webpage in the foreground, it works fine,
but if I switch it to background and switch it back, all frequency bars lose their signals.

I examined the code and found that the methods `getFloatFrequencyData` and `getByteFrequencyData` will
keep giving non-sense value when the webpage runs in the background.
The former will return negative values which smaller and smaller, and the latter will always return zero.
That’s the reason it doesn’t draw frequency bars.

After researching, I found no similar report about the issue on the Internet.
So I read the document about the AudioContext again. Maybe I miss something before.

From Mozilla’s [developer page](https://developer.mozilla.org/en-US/docs/Web/API/Web_Audio_API/Visualizations_with_Web_Audio_API),
I found a [life-saving demo](https://mdn.github.io/voice-change-o-matic/), and it doesn’t suffer the problem.
The major difference between the demo and my code is the demo add a gainNode during the connecting procedure,
to play the sound from the microphone.

Great.
I change my code in that way and do some slight modifications to keep the volume of the microphone at
a low level; it works.

```typescript
  const audioCtx = new AudioContext()
  const stream = await navigator.mediaDevices.getUserMedia(constraints)
  const analyser = audioCtx.createAnalyser()
  analyser.fftSize = 256
  analyser.minDecibels = -80
  analyser.maxDecibels = -10
  analyser.smoothingTimeConstant = 0.85

  const source = audioCtx.createMediaStreamSource(stream)
  const gainNode = audioCtx.createGain()
  // Reduce micphone's volume to 0.01 (not 0)
  // or safari will give non-sense data in getFloatFrequencyData() and getByteFrequencyData()
  // after switch mornin's page to background
  gainNode.gain.value = 0.01
  source.connect(analyser)
  // connect gainNode after analyzer to reduce the volume
  analyser.connect(gainNode)
  gainNode.connect(audioCtx.destination)
```

