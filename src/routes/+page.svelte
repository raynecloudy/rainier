<script lang="ts">
  import { onMount } from "svelte";
  // @ts-ignore
  import { version } from "/package.json";

  // @ts-ignore
  let videoElement: HTMLVideoElement = $state();

  let progress = $state(0);

  let showControls = $state(true);

  let showMore = $state(false);

  let autoplay = $state(false);
  let currentTime = $state(0);
  let duration = $state(0);
  let hideOptions = $state(true);
  let loop = $state(false);
  let mute = $state(false);
  let src: string | null = $state(null);
  let title = $state("");

  onMount(async () => {
    const params = new URLSearchParams(location.search);
    autoplay = params.get("autoplay") !== null;
    currentTime = parseInt(params.get("time") ?? "0");
    hideOptions = params.get("hideOptions") !== null;
    loop = params.get("loop") !== null;
    mute = params.get("mute") !== null;
    src = params.get("url");
    title = params.get("title") ?? "Untitled";

    if (!src) return;
    const res = await fetch(src);
    if (!res.ok) return;
    if (res.headers.get("Content-Type") === "application/json") {
      const data = await res.json();
      if (data.autoplay) autoplay = data.autoplay;
      if (data.hideOptions) hideOptions = data.hideOptions;
      if (data.loop) loop = data.loop;
      if (data.mute) mute = data.mute;
      if (data.time) currentTime = data.time;
      if (data.title) title = data.title;
      if (data.url) src = data.url;
    }

    if (autoplay) {
      videoElement.play();
    }
  });

  const updateProgress = () => (currentTime / duration) * 100;

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
    --overlay: #2b2b2b5e;
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
    text-align: center;
    flex-direction: column;
    padding: 1rem;
    gap: 1rem;
  }
  
  controls {
    width: 40vw;
    left: 50vw;
    bottom: 1rem;
    transform: translateX(-50%);
    opacity: 0;
    filter: blur(0.2rem);
    transition: opacity 2s 0.5s linear, filter 2s 0.5s linear;
  }
  
  controls:is(.show, :has(:focus-visible)) {
    opacity: 1;
    filter: blur(0rem);
    transition: opacity 0.4s linear;
  }
  
  controls > div {
    display: flex;
    gap: 1rem;
    width: 100%;
    align-items: center;
  }
  
  more {
    width: calc(20vw - 1rem);
    left: 50vw;
    bottom: 7rem;
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

  button:is(:hover, :focus-visible):not(:active) {
    outline: none;
    filter: brightness(3);
  }

  more button {
    text-align: inherit;
  }

  .time {
    font-family: monospace;
  }

  a {
    color: inherit;
  }

  controls b {
    max-width: 100%;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }
</style>

<svelte:body onmouseleave={() => showControls = videoElement.paused || showMore ? true : false} onmouseenter={() => showControls = true} onmousemove={() => showControls = true} oncontextmenu={(event) => { event.preventDefault(); showMore = !showMore; showControls = true; }}></svelte:body>

<video src={src} bind:this={videoElement} bind:currentTime={currentTime} bind:duration={duration} bind:muted={mute} loop={loop} ontimeupdate={() => { if (progress !== updateProgress() && !showMore) showControls = false; progress = updateProgress(); }} onended={() => showControls = true} onclick={controlVideo} disablepictureinpicture>
  <track kind="captions">
</video>
{#if videoElement}
  <controls class:show={showControls}>
    <b title={title}>{title}</b>
    <div>
      <button aria-label={videoElement.paused ? videoElement.ended ? "Replay" : "Play" : "Pause"} onclick={controlVideo}><i class="fa-solid fa-{videoElement.paused ? videoElement.ended ? "arrow-rotate-left" : "play" : "pause"}"></i></button>
      <div class="time">{Math.round(currentTime)}</div>
      <progressbar>
        <div style:width={`${progress}%`}></div>
      </progressbar>
      <div class="time">{Math.round(duration)}</div>
      {#if !hideOptions}
        <button aria-label="More options" onclick={() => showMore = !showMore}><i class="fa-solid fa-ellipsis-vertical"></i></button>
      {/if}
    </div>
  </controls>
  {#if !hideOptions}
    <more class:show={showMore} inert={!showMore}>
      <button onclick={() => videoElement.currentTime = 0}><i class="fa-solid fa-arrow-left"></i> Restart</button>
      <button onclick={() => loop = !loop}><i class="fa-solid fa-arrow-rotate-right"></i> Loop ({loop ? "on" : "off"})</button>
      <button onclick={() => mute = !mute}><i class="fa-solid fa-volume-xmark"></i> Mute ({mute ? "on" : "off"})</button>
      <sub><a href="https://github.com/raynecloudy/rainier" target="_top">Rainier</a> v{version}</sub>
    </more>
  {/if}
{/if}
