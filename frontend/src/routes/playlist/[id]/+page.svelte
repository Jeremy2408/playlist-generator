<script lang="ts">
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabaseClient';
  import { page } from '$app/stores';
  import { get } from 'svelte/store';
  import { goto } from '$app/navigation';

  let playlistId = '';
  let accessToken = '';
  let tracks: any[] = [];
  let error = '';
  let playlistName = '';

  function formatDuration(ms: number) {
    const minutes = Math.floor(ms / 60000);
    const seconds = Math.floor((ms % 60000) / 1000).toString().padStart(2, '0');
    return `${minutes}:${seconds}`;
  }

  onMount(async () => {
    playlistId = get(page).params.id;

    const { data } = await supabase.auth.getSession();
    const session = data.session;

    if (!session) {
      if (typeof window !== 'undefined') {
        window.location.href = '/login';
      }
      return;
    }

    accessToken = session.provider_token || session.access_token || '';

    try {
      const res = await fetch(`https://api.spotify.com/v1/playlists/${playlistId}/tracks`, {
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
      });

      const json = await res.json();
      tracks = json.items.map((item: any) => item.track);

      const metaRes = await fetch(`https://api.spotify.com/v1/playlists/${playlistId}`, {
        headers: {
          Authorization: `Bearer ${accessToken}`,
        },
      });
      const meta = await metaRes.json();
      playlistName = meta.name;
    } catch (err: any) {
      error = 'Failed to load playlist: ' + err.message;
    }
  });
</script>

<div class="max-w-2xl mx-auto mt-10 px-4">
  <button class="mb-4 text-blue-600 hover:underline" on:click={() => goto('/dashboard')}>
    ← Back to Dashboard
  </button>

  <h2 class="text-3xl font-extrabold mb-6 text-center">{playlistName}</h2>

  {#if tracks.length > 0}
    <ul class="space-y-4">
      {#each tracks as track}
        <li class="border p-4 rounded shadow-sm bg-white flex items-center gap-4">
          {#if track.album?.images?.[0]?.url}
            <img src={track.album.images[0].url} alt="album art" class="w-16 h-16 object-cover rounded" />
          {/if}
          <div>
            <p class="text-lg font-bold">{track.name}</p>
            <p class="text-sm text-gray-600">{track.artists[0]?.name}</p>
            <p class="text-sm text-gray-500">{formatDuration(track.duration_ms)}</p>
          </div>
        </li>
      {/each}
    </ul>
  {:else if error}
    <p class="text-center text-red-500 mt-6">{error}</p>
  {:else}
    <p class="text-center text-gray-500 mt-6">Loading...</p>
  {/if}
</div>
