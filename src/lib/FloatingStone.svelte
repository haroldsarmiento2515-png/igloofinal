<script>
  export let variant = 'glow';
  export let reducedMotion = false;
  export let exitProgress = 0;
  export let appearProgress = 1;
</script>

<button
  class={`stone ${variant}`}
  style={`--exit:${exitProgress}; --appear:${appearProgress};`}
  type="button"
  aria-label="Enter the stone"
>
  <span class="stone-core" />
</button>

<style>
  .stone {
    position: relative;
    width: clamp(120px, 18vw, 180px);
    aspect-ratio: 1;
    border-radius: 40% 60% 45% 55%;
    border: none;
    background: radial-gradient(circle at 30% 30%, #eff4f8, #aeb7c1 65%, #8c949d 100%);
    box-shadow: 0 20px 60px rgba(20, 28, 36, 0.35);
    cursor: pointer;
    transform: translate3d(0, calc(var(--exit) * -140px), 0) scale(calc(0.85 + var(--appear) * 0.15));
    opacity: var(--appear);
    transition: transform 0.4s ease, box-shadow 0.4s ease;
  }

  .stone:hover {
    box-shadow: 0 28px 70px rgba(20, 28, 36, 0.45);
  }

  .stone-core {
    position: absolute;
    inset: 18% 22% 28% 25%;
    border-radius: 45% 55% 50% 50%;
    background: rgba(255, 255, 255, 0.45);
    filter: blur(0.5px);
  }

  .stone.glow {
    animation: float 4s ease-in-out infinite, spin 18s linear infinite;
  }

  .stone.etched {
    background: radial-gradient(circle at 35% 30%, #e6eef5, #b1bac5 55%, #838b95 100%);
    box-shadow: 0 22px 50px rgba(15, 22, 30, 0.4);
    animation: float 5s ease-in-out infinite, spin 22s linear infinite;
  }

  :global(.reduced-motion) .stone {
    animation: none;
  }

  @keyframes float {
    0%,
    100% {
      transform: translate3d(0, calc(var(--exit) * -140px + 12px), 0) scale(calc(0.85 + var(--appear) * 0.15));
    }
    50% {
      transform: translate3d(0, calc(var(--exit) * -140px - 12px), 0) scale(calc(0.85 + var(--appear) * 0.15));
    }
  }

  @keyframes spin {
    from {
      rotate: 0deg;
    }
    to {
      rotate: 360deg;
    }
  }
</style>
