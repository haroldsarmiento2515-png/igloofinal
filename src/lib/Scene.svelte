<script>
  import { onMount, onDestroy } from 'svelte';
  import { Canvas, T } from '@threlte/core';
  import { gsap } from 'gsap';
  import { ScrollTrigger } from 'gsap/ScrollTrigger';
  import Igloo from './Igloo.svelte';
  import Snow from './Snow.svelte';
  import Stone from './Stone.svelte';
  import CameraRig from './CameraRig.svelte';

  gsap.registerPlugin(ScrollTrigger);

  let container;
  let scrollProgress = 0;
  let portalProgress = 0;
  let modalOpen = false;

  const stones = [
    {
      name: 'Obsidian shard',
      color: '#1f2329',
      roughness: 0.35,
      metalness: 0.2,
      scale: 1.1,
      distortion: 0.35
    },
    {
      name: 'Quartz bloom',
      color: '#d8dfe8',
      roughness: 0.25,
      metalness: 0.05,
      scale: 1.05,
      distortion: 0.2
    },
    {
      name: 'Glacial basalt',
      color: '#38414b',
      roughness: 0.55,
      metalness: 0.1,
      scale: 1.2,
      distortion: 0.45
    }
  ];

  const clamp = (value, min = 0, max = 1) => Math.min(max, Math.max(min, value));

  let heroMix = 0;
  let interiorMix = 0;
  let galleryMix = 0;
  let galleryOffset = 0;

  const updateScrollValues = () => {
    heroMix = clamp(1 - scrollProgress * 2.4);
    interiorMix = clamp((scrollProgress - 0.22) / 0.28);
    galleryMix = clamp((scrollProgress - 0.52) / 0.48);
    galleryOffset = galleryMix * (stones.length - 1);
  };

  const openPortal = () => {
    if (modalOpen) return;
    modalOpen = true;
    gsap.to({ value: portalProgress }, {
      value: 1,
      duration: 1.2,
      ease: 'power3.inOut',
      onUpdate() {
        portalProgress = this.targets()[0].value;
      }
    });
  };

  const closePortal = () => {
    if (!modalOpen) return;
    modalOpen = false;
    gsap.to({ value: portalProgress }, {
      value: 0,
      duration: 1,
      ease: 'power2.out',
      onUpdate() {
        portalProgress = this.targets()[0].value;
      }
    });
  };

  onMount(() => {
    const trigger = ScrollTrigger.create({
      trigger: container,
      start: 'top top',
      end: 'bottom bottom',
      scrub: 1,
      onUpdate: (self) => {
        scrollProgress = self.progress;
        updateScrollValues();
      }
    });

    updateScrollValues();

    return () => {
      trigger.kill();
    };
  });

  onDestroy(() => {
    ScrollTrigger.getAll().forEach((instance) => instance.kill());
  });
</script>

<div class="scene" bind:this={container}>
  <div class="canvas-wrap" aria-hidden="true">
    <Canvas dpr={[1, 1.8]} shadows>
      <T.PerspectiveCamera makeDefault position={[0, 1.6, 9]} fov={42} near={0.1} far={40} />
      <T.Fog color={'#2e343b'} near={6} far={18} />
      <T.AmbientLight intensity={0.65} />
      <T.DirectionalLight position={[4, 6, 4]} intensity={1.1} />
      <T.DirectionalLight position={[-6, 4, -4]} intensity={0.55} color={'#c6d1dc'} />

      <CameraRig {scrollProgress} {portalProgress} />

      <group position={[0, -0.6, 0]}>
        <Igloo {heroMix} {interiorMix} />
        <Snow {heroMix} />
      </group>

      {#each stones as stone, index}
        <Stone
          {...stone}
          index={index}
          offset={galleryOffset}
          interiorMix={interiorMix}
          galleryMix={galleryMix}
          on:select={openPortal}
        />
      {/each}
    </Canvas>
  </div>

  <div class="overlay">
    <section class="panel hero" style={`opacity: ${heroMix};`}>
      <p class="eyebrow">Blueprint Igloo</p>
      <h1>Carved in quiet winter light.</h1>
      <p class="detail">Scroll to dive below the surface and into the core.</p>
    </section>

    <section class="panel interior" style={`opacity: ${interiorMix};`}>
      <p class="eyebrow">Within the Igloo</p>
      <h2>The first stone hovers in a hushed gray void.</h2>
      <p class="detail">Click the stone to step inside its crystalline grain.</p>
    </section>

    <section class="panel gallery" style={`opacity: ${galleryMix};`}>
      <p class="eyebrow">Gallery of Stones</p>
      <h2>Geometries rise and fall in continuous procession.</h2>
      <p class="detail">Each form carries a different mineral memory.</p>
    </section>
  </div>

  {#if modalOpen}
    <div class="modal" role="dialog" aria-modal="true" aria-label="Inside the stone">
      <div class="modal-glass">
        <p class="eyebrow">Inside the Core</p>
        <h3>Light fractures through crystalline layers.</h3>
        <p>
          The camera is now embedded in the stone surface, revealing the luminous
          seams that power the igloo.
        </p>
        <button type="button" on:click={closePortal}>Return to the chamber</button>
      </div>
    </div>
  {/if}
</div>

<style>
  .scene {
    position: relative;
    min-height: 360vh;
    background: linear-gradient(180deg, #dfe7f1 0%, #b6bec7 45%, #707780 100%);
    color: #1b2127;
  }

  .canvas-wrap {
    position: sticky;
    top: 0;
    height: 100vh;
    width: 100%;
  }

  :global(canvas) {
    display: block;
    width: 100%;
    height: 100%;
  }

  .overlay {
    position: sticky;
    inset: 0;
    pointer-events: none;
  }

  .panel {
    position: absolute;
    left: 8vw;
    max-width: 420px;
    transition: opacity 0.3s ease;
  }

  .panel.hero {
    top: 18vh;
  }

  .panel.interior {
    top: 36vh;
  }

  .panel.gallery {
    top: 58vh;
  }

  .eyebrow {
    text-transform: uppercase;
    letter-spacing: 0.5em;
    font-size: 0.65rem;
    margin-bottom: 0.8rem;
    color: #404851;
  }

  h1,
  h2,
  h3 {
    margin: 0 0 1rem;
    font-weight: 600;
  }

  .detail {
    font-size: 0.95rem;
    color: #2d343c;
  }

  .modal {
    position: fixed;
    inset: 0;
    display: grid;
    place-items: center;
    background: rgba(8, 12, 16, 0.4);
    backdrop-filter: blur(8px);
  }

  .modal-glass {
    padding: 2.5rem 2.8rem;
    border-radius: 20px;
    border: 1px solid rgba(255, 255, 255, 0.2);
    background: linear-gradient(130deg, rgba(255, 255, 255, 0.4), rgba(255, 255, 255, 0.05));
    color: #0c1116;
    max-width: 420px;
    box-shadow: 0 30px 80px rgba(0, 0, 0, 0.3);
    text-align: left;
  }

  .modal-glass button {
    margin-top: 1.5rem;
  }

  @media (max-width: 768px) {
    .panel {
      left: 8vw;
      right: 8vw;
      max-width: none;
    }

    .modal-glass {
      margin: 0 1.5rem;
    }
  }
</style>
