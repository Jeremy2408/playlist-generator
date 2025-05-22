<script lang="ts">
  import { supabase } from '$lib/supabaseClient';

  let email = '';
  let password = '';
  let message = '';

  async function login() {
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    if (!error) window.location.href = '/dashboard';
    else message = error.message;
  }

  async function signup() {
    const { error } = await supabase.auth.signUp({ email, password });
    message = error ? error.message : 'Signup success! Check your email.';
  }

  async function loginWithSpotify() {
    await supabase.auth.signInWithOAuth({
      provider: 'spotify',
      options: {
        redirectTo: 'http://localhost:5173/dashboard'
      }
    });
  }
</script>

<div class="max-w-md mx-auto mt-10 space-y-4">
  <input
    type="email"
    bind:value={email}
    class="w-full p-2 border rounded"
    placeholder="Email" />

  <input
    type="password"
    bind:value={password}
    class="w-full p-2 border rounded"
    placeholder="Password" />

  <button on:click={login} class="bg-blue-500 text-white px-4 py-2 rounded w-full">Login</button>
  <button on:click={signup} class="bg-green-500 text-white px-4 py-2 rounded w-full">Sign Up</button>

  <button on:click={loginWithSpotify} class="bg-black text-white px-4 py-2 rounded w-full">
    Login with Spotify
  </button>

  {#if message}
    <p class="text-center text-sm mt-2">{message}</p>
  {/if}
</div>


