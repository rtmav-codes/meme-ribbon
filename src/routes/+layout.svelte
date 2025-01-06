<script lang="ts">
	import Nav from '$lib/components/Nav.svelte';
	import Footer from '../lib/components/Footer.svelte';
	import '../app.css';
</script>

<div class="ribbon-container overflow-hidden">

	{#each Array(30) as _, i}
		<div 
			class="ribbon" 
			style="animation-delay: {i * 0.5}s; left: {Math.random() * 100}%"
		></div>
	{/each}

</div>
	
	<Nav />
	
	<slot />
	
	<Footer />

<style>
	.ribbon-container {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		pointer-events: none;
		z-index: 50;
	}

	.ribbon {
		position: absolute;
		top: -50px;
		width: 36px;
		height: 36px;
		background-image: url('/ribbon.png');
		background-size: cover;
		background-position: center;
		opacity: 0.6;
		border-radius: 4px;
		animation: fall 10s linear infinite;
	}

	@keyframes fall {
		0% {
			transform: translateY(-20px) rotate(0deg);
		}
		100% {
			transform: translateY(100vh) rotate(720deg);
		}
	}
</style>