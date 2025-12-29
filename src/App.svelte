<script>
  import { onMount } from 'svelte';
  import Scene from './lib/Scene.svelte';

  let reducedMotion = false;
  let webglSupported = true;
  let ready = false;

  const checkWebglSupport = () => {
    const canvas = document.createElement('canvas');
    return Boolean(canvas.getContext('webgl2') || canvas.getContext('webgl'));
  };

  onMount(() => {
    const motionQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    reducedMotion = motionQuery.matches;
    webglSupported = checkWebglSupport();
    ready = true;

    const handleChange = (event) => {
      reducedMotion = event.matches;
    };

    motionQuery.addEventListener('change', handleChange);

    return () => {
      motionQuery.removeEventListener('change', handleChange);
    };
  });
</script>

<main class:reduced-motion={reducedMotion}>
  {#if !ready}
    <section class="loading">
      <p>Preparing the glacial experience...</p>
    </section>
  {:else if reducedMotion || !webglSupported}
    <section class="fallback">
      <img src="/fallback-igloo.svg" alt="Illustrated igloo under arctic light" />
      <div>
        <p class="eyebrow">Igloo / Stillness</p>
        <h1>The glacial experience, distilled.</h1>
        <p>
          Your device is set to reduce motion or does not support WebGL. Here is a
          still preview of the 3D story.
        </p>
      </div>
    </section>
  {:else}
    <Scene />
  {/if}
</main>
