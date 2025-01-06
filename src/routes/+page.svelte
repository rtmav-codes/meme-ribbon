<!-- Landing/Splash Page -->

<!-- Main container with gradient background -->
<div class="h-screen w-screen bg-[radial-gradient(ellipse_at_center,_var(--tw-gradient-stops))] from-rose-300 via-pink-300 to-pink-100 text-white p-4 flex flex-col items-center justify-center">
    <!-- Logo/Emoji Display -->
    <h1 class="text-7xl">🎀</h1>
    
    <!-- Continue Button -->
    <a href="/home" class="btn bg-white lg:text-xl text-lg font-bold text-[#f35087] mt-10">Continue</a>
</div>

<!-- Svelte script section -->
<script lang="ts">
    import { onMount } from 'svelte';

    // Component initialization
    onMount(() => {
        console.log('Hello World');
    }); 
</script>