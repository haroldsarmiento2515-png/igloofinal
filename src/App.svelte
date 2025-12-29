<script>
  import IglooHero from './lib/IglooHero.svelte';
  import InteriorScene from './lib/InteriorScene.svelte';
  import FloatingStone from './lib/FloatingStone.svelte';
  import SnowParticles from './lib/SnowParticles.svelte';
  import StoneModal from './lib/StoneModal.svelte';
  import Counter from './lib/Counter.svelte';

  import './app.css';  // global styles
</script>

<main>
  <IglooHero />
  <InteriorScene />
  <FloatingStone />
  <SnowParticles />
  <StoneModal />
  <Counter />
</main>

<style>
  main {
    margin-top: 70px; /* adjust if you add fixed nav later */
  }
</style>