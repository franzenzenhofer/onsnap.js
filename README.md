#Snap.js - creating an onSnap event

the idea is simple: throw a snap event is someone snaps. simple enough. and it works like this:

some constants to make the code more clear.
this is the overall (over the whole frequency) "loudness" we are looking for

    THRESHOLD = 50000

and the minimum time we wait until we listen for a snap again

    TIME_UNTIL_LISTEN_AGAIN = 1000

uh oh, a global variable, if we are in listening mode, or not

    listen = true

an ugly implementation to sum all numbers of an array
it's ugly, because the UInt8Array does not have a reduce function

    sum = (ar) ->
      s = 0
      n = ar.length
      while(n--)
        s = s+ar[n]
      return s

after all is said and done, ans somebody snapped, let's throw a snap event.

    throwSnapEvent = () -> 
      event = new CustomEvent("snap", {
        bubbles: true,
        cancelable: false
      })
      window.document.body.dispatchEvent(event)

basically we plug an audio stream into the method
and handly somehting more useable to a watch loop

    gotStream = (stream) ->
      context = new webkitAudioContext()
      analyser = context.createAnalyser()
      source = context.createMediaStreamSource( stream )
      source.connect(analyser);
      watchLoop(analyser)

the watchLoop calls itself over and over again (that's why it 's called a loop')
and handles the all the date to an analysing function

    watchLoop = (analyser) ->
      loop_id = window.webkitRequestAnimationFrame((()->
        watchLoop(analyser)))

      freqByteData = new Uint8Array(analyser.frequencyBinCount)
      analyser.getByteFrequencyData(freqByteData)
      if listen
        lookForSnap(freqByteData)

behold, the very, very sophisticated analysing
yes, we just make a sum over the whole frequency array

    lookForSnap = (freqData) ->
      fsum = sum(freqData)

if it's loud, then throw the snap event

      if (fsum > THRESHOLD)
        throwSnapEvent()

but the thing is, we only need one snap event, so we wait some time, until we listen again

        listen = false
        window.setTimeout((()->listen=true), TIME_UNTIL_LISTEN_AGAIN)

## initalize the whole logic
and trigger the nagbar

    navigator.webkitGetUserMedia( {audio:true}, gotStream )
    #document.addEventListener("snap", (()->alert("snap")))