<script>
  import { createEventDispatcher, onMount } from 'svelte';

  const dispatch = createEventDispatcher();

  const close = () => dispatch('close');

  const onKeydown = (event) => {
    if (event.key === 'Escape') {
      close();
    }
  };

  onMount(() => {
    document.addEventListener('keydown', onKeydown);
    return () => document.removeEventListener('keydown', onKeydown);
  });
</script>

<div class="overlay" role="dialog" aria-modal="true">
  <div class="modal">
    <button class="close" type="button" on:click={close}>Close</button>
    <h2>Inside the stone</h2>
    <p>
      The igloo holds a quiet core. Each stone keeps the warmth of the winter light,
      waiting for the next traveler to listen.
    </p>
  </div>
</div>

<style>
  .overlay {
    position: fixed;
    inset: 0;
    background: rgba(15, 18, 22, 0.65);
    display: grid;
    place-items: center;
    z-index: 10;
    backdrop-filter: blur(6px);
  }

  .modal {
    width: min(90vw, 420px);
    background: #f2f4f7;
    color: #1c2228;
    padding: 2rem;
    border-radius: 1.5rem;
    box-shadow: 0 30px 80px rgba(0, 0, 0, 0.35);
    display: grid;
    gap: 1rem;
  }

  .close {
    justify-self: flex-end;
    border: none;
    background: #1c2228;
    color: #f8fafc;
    padding: 0.5rem 1rem;
    border-radius: 999px;
    cursor: pointer;
  }

  .close:hover {
    background: #000;
  }
</style>
