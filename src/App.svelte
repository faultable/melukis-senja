<script>
  import { onMount } from 'svelte'

  import Analyser from './Analyser.svelte'

  let lyrics = []
  let isShowCredit = false
  let totalDuration = 4 * 60 - 3
  let currentTime = 0
  let determinedIndex = 0
  let songTitle = 'Budi Doremi - Melukis Senja'

  function constructLyrics(rawLyrics) {
    const parsedLyrics = rawLyrics.split('\n')
    const tsRegex = /\[(\d+):(\d+)]/g

    for (let i = 0; i < parsedLyrics.length; i++) {
      const line = parsedLyrics[i]
      const ts = tsRegex.exec(line)

      if (ts) {
        const text = line.replace(tsRegex, '').trim()

        if (text) {
          const minutes = ts[1] * 60 * 1000
          const seconds = ts[2] * 1000

          lyrics.push({
            ts: minutes + seconds,
            text,
          })
        }
      }
    }
  }

  onMount(() => {
    window
      .fetch('/lirik.txt')
      .then((res) => res.text())
      .then((res) => {
        constructLyrics(res)
      })

    setTimeout(() => {
      document.body.style.opacity = 1
    }, 1000)
  })

  function updateCurrentTime(time) {
    currentTime = Math.floor(time * 1000)

    checkLyrics(currentTime)
  }

  function checkLyrics(time) {
    determinedIndex = lyrics.findIndex(({ ts }) => time <= ts)
  }

  function showCredit() {
    document.getElementsByTagName('canvas')[0].style.filter = 'blur(5px)'

    setTimeout(() => {
      isShowCredit = true
    }, 1000)
  }
</script>

<div>
  <div class="o-lyrics">
    {#if lyrics.length > 0 && determinedIndex !== -1}
      <h2>{lyrics[determinedIndex].text}</h2>
    {/if}

    {#if isShowCredit}
      <p>
        <span>Song by {songTitle}</span>
        <a href="https://faultable.dev">@faultable</a>
      </p>
    {/if}
  </div>
  <Analyser
    audio="/music.mp3"
    {totalDuration}
    {updateCurrentTime}
    {showCredit} />
</div>
