#onSnap.js - creating a Snap event

the idea is simple: **throw a snap event as soon as someone snaps**. simple enough.
this is written litcoffee style (literate programming + coffeescript), so yes, this README.md is the acutal sourcecode.

lets start.

some **constants** to make the code more clear.

this is the overall (over the whole frequency) "loudness" we are looking for

    THRESHOLD = 50000

and the time we wait until we listen for a snap again (minimum time between snap events)

    TIME_UNTIL_LISTEN_AGAIN = 1000

uh oh, a **(in scope) global variable**, if we are in listening mode, or not

    _l_i_s_t_e_n_ = true

yeah, i make ugly things (like side effects) ugly. that's why this variable is so ugly. 

and while we are at the ugly topic, look: 
an ugly implementation to sum all numbers of an array
it's ugly, because the UInt8Array does not have a reduce function (not a joke)

    sum = (ar) ->
      s = 0
      n = ar.length
      while(n--)
        s = s+ar[n]
      return s

after all is said and done, and somebody snapped, let's throw a snap event.

    throwSnapEvent = () -> 
      SnapEvent = new CustomEvent("snap", {
        bubbles: true,
        cancelable: false
      })
      window.document.body.dispatchEvent(SnapEvent)

below **the thing we initalize as soon as we got an audio stream**.
basically we plug an audio stream into the method
and handly something more useable (an analyser based on the audio stream) to a watch loop

    gotStream = (stream) ->
      context = new webkitAudioContext()
      analyser = context.createAnalyser()
      source = context.createMediaStreamSource( stream )
      source.connect(analyser);
      watchLoop(analyser)

the **watchloop** calls itself over and over again (that's why it 's called a loop')
and handles the all the date to an analysing function

    watchLoop = (analyser) ->
      loop_id = window.webkitRequestAnimationFrame((()->
        watchLoop(analyser)))

      freqByteData = new Uint8Array(analyser.frequencyBinCount)
      analyser.getByteFrequencyData(freqByteData)
      if _l_i_s_t_e_n_
        lookForSnap(freqByteData)

behold, the very, very sophisticated **analysing function**.
yes, we just make a sum over the whole frequency array

    lookForSnap = (freqData) ->
      fsum = sum(freqData)

if it's loud, then throw the snap event

      if (fsum > THRESHOLD)

ok, you got me, basically we are listening for noise, so here comes the noise, we throw

        throwSnapEvent()

but the thing is, we only need one snap event, so we wait some time, until we listen again

        _l_i_s_t_e_n_ = false
        window.setTimeout((()->_l_i_s_t_e_n_=true), TIME_UNTIL_LISTEN_AGAIN)

## initalize the whole logic

and trigger the nagbar

    navigator.webkitGetUserMedia( {audio:true}, gotStream )
    #document.addEventListener("snap", (()->alert("snap")))

##compile this whole thing

    #coffee -c onsnap.litcoffee

creates the onsnap.js file.

actually onsnap.litcoffee is just a symblic link to this README.md.

##how to use this

    #<script src="onsnap.js"></script>
    #document.addEventListener("snap", ...);