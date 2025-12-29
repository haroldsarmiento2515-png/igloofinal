<script>
  import { onMount } from 'svelte';
  import SnowParticles from './SnowParticles.svelte';

  export let progress = 0;
  export let reducedMotion = false;

  let iglooEl;
  let targetX = 0;
  let targetY = 0;
  let currentX = 0;
  let currentY = 0;
  let rafId;

  const updateParallax = () => {
    currentX += (targetX - currentX) * 0.08;
    currentY += (targetY - currentY) * 0.08;
    if (iglooEl) {
      iglooEl.style.setProperty('--parallax-x', `${currentX}px`);
      iglooEl.style.setProperty('--parallax-y', `${currentY}px`);
    }
    rafId = requestAnimationFrame(updateParallax);
  };

  onMount(() => {
    if (reducedMotion) {
      return () => {};
    }

    const handleMove = (event) => {
      const { innerWidth, innerHeight } = window;
      const offsetX = (event.clientX / innerWidth - 0.5) * 30;
      const offsetY = (event.clientY / innerHeight - 0.5) * 30;
      targetX = offsetX;
      targetY = offsetY;
    };

    window.addEventListener('pointermove', handleMove);
    rafId = requestAnimationFrame(updateParallax);

    return () => {
      window.removeEventListener('pointermove', handleMove);
      cancelAnimationFrame(rafId);
    };
  });
</script>

<section class="hero" style={`--hero-progress:${progress};`}>
  <div class="sky">
    <div class="mountains">
      <span class="mountain" />
      <span class="mountain" />
      <span class="mountain" />
    </div>
    <SnowParticles {reducedMotion} />
  </div>
  <div class="igloo" bind:this={iglooEl}>
    <svg viewBox="0 0 400 260" class="igloo-svg" aria-hidden="true">
      <g class="igloo-trace">
        <path pathLength="100" d="M40 210 Q200 40 360 210" />
        <path pathLength="100" d="M85 210 Q200 85 315 210" />
        <path pathLength="100" d="M120 210 Q200 120 280 210" />
        <path pathLength="100" d="M160 210 Q200 150 240 210" />
        <path pathLength="100" d="M180 210 Q200 175 220 210" />
        <path pathLength="100" d="M60 210 H340" />
      </g>
      <g class="igloo-lights">
        <circle r="4" fill="#ffffff">
          <animateMotion dur="2.8s" repeatCount="1" path="M40 210 Q200 40 360 210" />
        </circle>
        <circle r="3" fill="#f7f7f7">
          <animateMotion dur="2.4s" repeatCount="1" begin="0.2s" path="M85 210 Q200 85 315 210" />
        </circle>
        <circle r="3" fill="#ffffff">
          <animateMotion dur="2.2s" repeatCount="1" begin="0.4s" path="M120 210 Q200 120 280 210" />
        </circle>
        <circle r="2.6" fill="#ffffff">
          <animateMotion dur="2s" repeatCount="1" begin="0.6s" path="M160 210 Q200 150 240 210" />
        </circle>
        <circle r="2.2" fill="#ffffff">
          <animateMotion dur="1.8s" repeatCount="1" begin="0.8s" path="M60 210 H340" />
        </circle>
      </g>
      <g class="igloo-solid">
        <path d="M40 210 Q200 40 360 210" />
        <path d="M85 210 Q200 85 315 210" />
        <path d="M120 210 Q200 120 280 210" />
        <path d="M160 210 Q200 150 240 210" />
        <path d="M180 210 Q200 175 220 210" />
        <path d="M60 210 H340" />
      </g>
    </svg>
  </div>
  <div class="hero-text">
    <p>Igloo</p>
    <h1>Carved in quiet winter light.</h1>
  </div>
</section>

<style>
  .hero {
    position: relative;
    width: 100%;
    height: 100%;
    display: grid;
    place-items: center;
    overflow: hidden;
  }

  .sky {
    position: absolute;
    inset: 0;
    background: linear-gradient(180deg, #dfe6ee 0%, #bfc7d1 55%, #d8dde4 100%);
  }

  .mountains {
    position: absolute;
    bottom: 15%;
    left: 0;
    right: 0;
    display: flex;
    justify-content: space-around;
    align-items: flex-end;
  }

  .mountain {
    width: 35vw;
    height: 26vh;
    max-width: 300px;
    max-height: 220px;
    background: linear-gradient(140deg, #b2bac4, #9098a1);
    clip-path: polygon(50% 0%, 100% 100%, 0% 100%);
    opacity: 0.6;
  }

  .mountain:nth-child(2) {
    width: 45vw;
    height: 32vh;
    max-width: 380px;
    opacity: 0.4;
  }

  .mountain:nth-child(3) {
    width: 28vw;
    height: 20vh;
    max-width: 260px;
    opacity: 0.5;
  }

  .igloo {
    position: relative;
    z-index: 2;
    width: min(70vw, 460px);
    transform: translate3d(var(--parallax-x, 0px), var(--parallax-y, 0px), 0);
    transition: transform 0.2s ease-out;
  }

  .igloo-svg {
    width: 100%;
    height: auto;
    overflow: visible;
  }

  .igloo-trace path {
    fill: none;
    stroke: #f5f7fb;
    stroke-width: 2.5;
    stroke-linecap: round;
    stroke-linejoin: round;
    stroke-dasharray: 100;
    stroke-dashoffset: 100;
    animation: trace 2.8s ease forwards;
  }

  .igloo-trace path:nth-child(2) {
    animation-delay: 0.1s;
  }

  .igloo-trace path:nth-child(3) {
    animation-delay: 0.2s;
  }

  .igloo-trace path:nth-child(4) {
    animation-delay: 0.3s;
  }

  .igloo-trace path:nth-child(5) {
    animation-delay: 0.4s;
  }

  .igloo-trace path:nth-child(6) {
    animation-delay: 0.2s;
  }

  .igloo-lights {
    opacity: 0.9;
  }

  .igloo-solid {
    opacity: 0;
    animation: solid 0.8s ease forwards;
    animation-delay: 2.7s;
  }

  .igloo-solid path {
    fill: none;
    stroke: #ffffff;
    stroke-width: 4;
    stroke-linecap: round;
    stroke-linejoin: round;
  }

  .hero-text {
    position: absolute;
    bottom: 12vh;
    text-align: center;
    color: #26313b;
    z-index: 3;
  }

  .hero-text p {
    text-transform: uppercase;
    letter-spacing: 0.4em;
    font-size: 0.7rem;
    margin-bottom: 0.5rem;
  }

  .hero-text h1 {
    font-size: clamp(1.6rem, 3vw, 2.6rem);
    margin: 0;
  }

  :global(.reduced-motion) .igloo-trace path,
  :global(.reduced-motion) .igloo-lights {
    animation: none;
    opacity: 0;
  }

  :global(.reduced-motion) .igloo-solid {
    opacity: 1;
    animation: none;
  }

  @keyframes trace {
    to {
      stroke-dashoffset: 0;
    }
  }

  @keyframes solid {
    to {
      opacity: 1;
    }
  }

  @media (max-width: 640px) {
    .mountain {
      width: 50vw;
    }

    .hero-text {
      padding: 0 1.5rem;
    }
  }
</style>
