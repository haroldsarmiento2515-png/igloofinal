<script>
  import { onMount } from 'svelte';
  import IglooHero from './lib/IglooHero.svelte';
  import InteriorScene from './lib/InteriorScene.svelte';
  import StoneModal from './lib/StoneModal.svelte';

  let scrollY = 0;
  let viewportHeight = 1;
  let reducedMotion = false;
  let modalOpen = false;
  let ticking = false;

  const clamp01 = (value) => Math.max(0, Math.min(1, value));

  const updateScroll = () => {
    scrollY = window.scrollY;
    ticking = false;
  };

  const onScroll = () => {
    if (!ticking) {
      window.requestAnimationFrame(updateScroll);
      ticking = true;
    }
  };

  const onResize = () => {
    viewportHeight = window.innerHeight;
  };

  onMount(() => {
    const media = window.matchMedia('(prefers-reduced-motion: reduce)');
    reducedMotion = media.matches;

    const handleMedia = (event) => {
      reducedMotion = event.matches;
    };

    viewportHeight = window.innerHeight;
    window.addEventListener('scroll', onScroll, { passive: true });
    window.addEventListener('resize', onResize);
    media.addEventListener('change', handleMedia);

    return () => {
      window.removeEventListener('scroll', onScroll);
      window.removeEventListener('resize', onResize);
      media.removeEventListener('change', handleMedia);
    };
  });

  $: sectionProgress = scrollY / viewportHeight;
  $: heroProgress = clamp01(sectionProgress);
  $: interiorSwitch = clamp01(sectionProgress - 1);
  $: interior1Opacity = clamp01(heroProgress - interiorSwitch);
  $: interior2Opacity = interiorSwitch;
  $: heroScale = 1 + heroProgress * 0.2;
  $: heroFade = 1 - heroProgress * 0.9;
  $: stoneExit = interiorSwitch;
  $: stoneAppear = interiorSwitch;
</script>

<svelte:window class:reduced-motion={reducedMotion} />

{#if modalOpen}
  <StoneModal on:close={() => (modalOpen = false)} />
{/if}

<div class="scroll-root">
  <div class="stage">
    <div class="hero-stage" style={`--scale:${heroScale}; --fade:${heroFade};`}>
      <IglooHero progress={heroProgress} {reducedMotion} />
    </div>
    <div class="interior-stage">
      <InteriorScene
        opacity={interior1Opacity}
        depth={heroProgress}
        stoneVariant="glow"
        {reducedMotion}
        stoneExitProgress={stoneExit}
        stoneAppearProgress={1}
        label="Inside the igloo"
        onStoneClick={() => (modalOpen = true)}
      />
      <InteriorScene
        opacity={interior2Opacity}
        depth={interiorSwitch}
        stoneVariant="etched"
        {reducedMotion}
        stoneExitProgress={0}
        stoneAppearProgress={stoneAppear}
        label="Deeper in the ice"
        onStoneClick={() => (modalOpen = true)}
      />
    </div>
  </div>
  <div class="scroll-spacer">
    <section class="scroll-section" />
    <section class="scroll-section" />
    <section class="scroll-section" />
  </div>
</div>
