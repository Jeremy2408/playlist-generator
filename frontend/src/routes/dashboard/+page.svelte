<script lang="ts">
  import { supabase } from '$lib/supabaseClient';
  import { onMount } from 'svelte';
  import type { User } from '@supabase/supabase-js';

  let user: User | null = null;

  onMount(async () => {
    const { data } = await supabase.auth.getSession();
    user = data?.session?.user ?? null;

    if (!user) {
      window.location.href = '/login';
    }
  });

  async function logout() {
    await supabase.auth.signOut();
    window.location.href = '/login';
  }
</script>

{#if user}
  <div class="mt-10 text-center">
    <h2 class="text-xl font-semibold">Welcome, {user.email}</h2>
    <button on:click={logout} class="bg-red-500 text-white px-4 py-2 mt-4 rounded">
      Logout
    </button>
    <p class="mt-6">🎧 Next: Display Spotify playlists here</p>
  </div>
{/if}
