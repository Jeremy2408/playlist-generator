<script>
  import '../app.css';
  import { onMount } from 'svelte';

  onMount(() => {
    if (typeof window !== 'undefined') {
      document.body.style.overflow = 'auto';
    }
  });
</script>

<div class="relative min-h-screen">
  <slot />
</div>
