<script>
  export let count = 60;
  export let reducedMotion = false;

  const particles = Array.from({ length: count }, (_, index) => {
    const size = Math.random() * 3 + 2;
    return {
      id: index,
      size,
      left: Math.random() * 100,
      delay: Math.random() * -20,
      duration: Math.random() * 10 + 12,
      opacity: Math.random() * 0.5 + 0.3
    };
  });
</script>

<div class="snow" aria-hidden="true">
  {#each particles as particle}
    <span
      class="flake"
      style={`--size:${particle.size}px; --left:${particle.left}%; --delay:${particle.delay}s; --duration:${particle.duration}s; --opacity:${particle.opacity};`}
    />
  {/each}
</div>

<style>
  .snow {
    position: absolute;
    inset: 0;
    overflow: hidden;
    pointer-events: none;
  }

  .flake {
    position: absolute;
    top: -10px;
    left: var(--left);
    width: var(--size);
    height: var(--size);
    border-radius: 50%;
    background: rgba(255, 255, 255, var(--opacity));
    animation: fall var(--duration) linear infinite;
    animation-delay: var(--delay);
  }

  :global(.reduced-motion) .flake {
    animation: none;
    top: 15%;
  }

  @keyframes fall {
    from {
      transform: translateY(-10px);
    }
    to {
      transform: translateY(110vh);
    }
  }
</style>
