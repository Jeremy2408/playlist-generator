<script lang="ts">
  import { supabase } from '$lib/supabaseClient';
  import { onMount } from 'svelte';
  import type { User } from '@supabase/supabase-js';

  let user: User | null = null;
  let accessToken = '';
  let playlists: any[] = [];
  let error = '';

  onMount(async () => {
    const { data } = await supabase.auth.getSession();
    const session = data.session;

    if (!session) {
      window.location.href = '/login';
      return;
    }

    user = session.user;
    // Spotify access token from the session
    accessToken = session.provider_token || session.access_token || '';

    if (!accessToken) {
      error = 'No Spotify access token found.';
      return;
    }

    try {
      const res = await fetch('https://api.spotify.com/v1/me/playlists', {
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
      });

      if (!res.ok) throw new Error(await res.text());

      const json = await res.json();
      playlists = json.items;
    } catch (err: any) {
      error = 'Failed to fetch playlists: ' + err.message;
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
  </div>

  {#if playlists.length > 0}
    <div class="mt-8 space-y-4">
      <h2 class="text-xl font-semibold text-center">Your Spotify Playlists</h2>
<ul class="grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 gap-6 mt-6">
  {#each playlists as playlist}
 <li class="bg-white p-4 rounded shadow hover:shadow-lg transition text-center">
      <img
      src={playlist.images[0]?.url}
      alt="cover"
      class="w-32 h-32 object-cover rounded mx-auto mb-2"
      style="width: 128px; height: 128px"

    />

      <p class="font-bold text-sm truncate">{playlist.name}</p>
      <p class="text-xs text-gray-500">{playlist.tracks.total} tracks</p>
    </li>
  {/each}
</ul>

    </div>
  {:else if error}
    <p class="text-red-600 mt-4">{error}</p>
  {:else}
    <p class="text-center mt-8 text-gray-500">Loading playlists...</p>
  {/if}
{/if}
