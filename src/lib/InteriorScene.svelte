<script>
  import FloatingStone from './FloatingStone.svelte';

  export let opacity = 1;
  export let depth = 0;
  export let stoneVariant = 'glow';
  export let reducedMotion = false;
  export let stoneExitProgress = 0;
  export let stoneAppearProgress = 1;
  export let onStoneClick = null;
  export let label = 'Within the igloo';
</script>

<section class="interior" style={`--opacity:${opacity}; --depth:${depth};`}>
  <div class="interior-backdrop" />
  <div class="content">
    <p>{label}</p>
    <FloatingStone
      variant={stoneVariant}
      {reducedMotion}
      exitProgress={stoneExitProgress}
      appearProgress={stoneAppearProgress}
      on:click={onStoneClick}
    />
  </div>
</section>

<style>
  .interior {
    position: absolute;
    inset: 0;
    display: grid;
    place-items: center;
    opacity: var(--opacity);
    transform: translate3d(0, calc(var(--depth) * 40px), 0) scale(calc(0.9 + var(--depth) * 0.1));
    transition: opacity 0.4s ease, transform 0.4s ease;
  }

  .interior-backdrop {
    position: absolute;
    inset: 0;
    background: #b5b9be;
  }

  .content {
    position: relative;
    display: grid;
    place-items: center;
    gap: 1.5rem;
    text-align: center;
    color: #1e242b;
    z-index: 1;
  }

  .content p {
    text-transform: uppercase;
    letter-spacing: 0.35em;
    font-size: 0.7rem;
  }
</style>
