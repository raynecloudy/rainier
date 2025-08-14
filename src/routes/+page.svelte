<script lang="ts">
  import { onMount } from "svelte";
  type Param = string | null;
  let autoplay: Param = $state(null);
  let src: Param = $state(null);
  let title: Param = $state(null);

  // @ts-ignore
  let videoElement: HTMLVideoElement = $state();

  let progress = $state(0);

  let showControls = $state(true);
  
  let showMore = $state(false);

  let currentTime = $state(0);
  let duration = $state(0);
  
  onMount(() => {
    const params = new URLSearchParams(location.search);
    autoplay = params.get("autoplay");
    src = params.get("src");
    title = params.get("title");

    if (autoplay) {
      videoElement.play();
    }
  });

  const updateProgress = () => {
    if (!videoElement) return 0;
    currentTime = videoElement.currentTime;
    duration = videoElement.duration;
    return (currentTime / duration) * 100;
  }

  const controlVideo = () => {
    if (videoElement.ended) videoElement.currentTime = 0;
    if (videoElement.paused) videoElement.play();
    else videoElement.pause();
    showControls = true;
  };
</script>

<style>
  @font-face {
    font-family: "Rubik";
    src: url("/Rubik.ttf");
  }

  :root {
    --overlay: #2b2b2b23;
    --overlay2: #dddddd36;
  }

  :global(body) {
    background-color: #000000;
    margin: 0;
    font-family: "Rubik";
    overflow: hidden;
    width: 100vw;
    height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    user-select: none;
    line-height: 1rem;
  }

  :global(body:not(:has(controls.show))) {
    cursor: none;
  }

  video {
    width: 100vw;
    height: 100vh;
  }

  controls, more {
    background-color: var(--overlay);
    color: #ffffff;
    backdrop-filter: blur(10rem);
    border-radius: 1rem;
    position: absolute;
    display: flex;
    padding: 1rem;
    gap: 1rem;
  }
  
  controls {
    width: 40vw;
    left: 50vw;
    bottom: 1rem;
    transform: translateX(-50%);
    align-items: center;
    opacity: 0;
    filter: blur(0.2rem);
    transition: opacity 2s 0.5s linear, filter 2s 0.5s linear;
  }
  
  controls.show {
    opacity: 1;
    filter: blur(0rem);
    transition: opacity 0.4s linear;
  }
  
  more {
    width: calc(20vw - 1rem);
    left: 50vw;
    bottom: 6rem;
    flex-direction: column;
    opacity: 0;
    transition: opacity 0.2s;
  }
  
  more.show {
    opacity: 1;
  }

  progressbar {
    background-color: var(--overlay2);
    display: block;
    height: 0.4rem;
    border-radius: 0.2rem;
    overflow: hidden;
    flex: 1 1;
  }
  
  progressbar div {
    height: 100%;
    background-color: #ffffff;
    transition: width 0.4s linear;
  }

  button {
    background-color: var(--overlay2);
    border: none;
    color: inherit;
    font: inherit;
    padding: 0.5rem;
    margin: 0;
    display: block;
    border-radius: 0.5rem;
    cursor: pointer;
    height: 2lh;
    min-width: 2lh;
  }

  more button {
    text-align: inherit;
  }

  .time {
    font-family: monospace;
  }
</style>

<svelte:body onmouseleave={() => showControls = videoElement.paused || showMore ? true : false} onmouseenter={() => showControls = true} onmousemove={() => showControls = true} oncontextmenu={(event) => { event.preventDefault(); showMore = !showMore; showControls = true; }}></svelte:body>

<video src={src} bind:this={videoElement} onloadeddata={() => duration = videoElement.duration} ontimeupdate={() => { if (progress !== updateProgress() && !showMore) showControls = false; progress = updateProgress(); }} onended={() => showControls = true} onclick={controlVideo} disablepictureinpicture>
  <track kind="captions">
</video>
{#if videoElement}
  <more class:show={showMore} inert={!showMore}>
    <button onclick={() => videoElement.currentTime = 0}><i class="fa-solid fa-arrow-left"></i> Restart</button>
    <button onclick={() => videoElement.loop = !videoElement.loop}><i class="fa-solid fa-arrow-rotate-right"></i> Loop ({videoElement.loop ? "on" : "off"})</button>
  </more>
  <controls class:show={showControls}>
    <div class="time">{Math.round(currentTime)}</div>
    <progressbar>
      <div style:width={`${progress}%`}></div>
    </progressbar>
    <div class="time">{Math.round(duration)}</div>
    <button aria-label={videoElement.paused ? videoElement.ended ? "Replay" : "Play" : "Pause"} onclick={controlVideo}><i class="fa-solid fa-{videoElement.paused ? videoElement.ended ? "arrow-rotate-left" : "play" : "pause"}"></i></button>
    <button aria-label="More options" onclick={() => showMore = !showMore}><i class="fa-solid fa-ellipsis-vertical"></i></button>
  </controls>
{/if}
