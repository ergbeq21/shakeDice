<script>
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	let number = '-';
	let lastShake = 0;
	const SHAKE_THRESHOLD = 15; 
	const SHAKE_COOLDOWN = 1000; 

	function roll() {
		number = Math.floor(Math.random() * 6) + 1;
	}

	function handleMotion(event) {
		if (!event.accelerationIncludingGravity) return;

		const acc = event.accelerationIncludingGravity;
		const totalAcc = Math.sqrt(
			(acc.x ?? 0) ** 2 +
			(acc.y ?? 0) ** 2 +
			(acc.z ?? 0) ** 2
		);

		const now = Date.now();
		if (totalAcc > SHAKE_THRESHOLD && now - lastShake > SHAKE_COOLDOWN) {
			lastShake = now;
			roll();
		}
	}

	onMount(() => {
		if (browser) {
			window.addEventListener('devicemotion', handleMotion);
		}
		return () => {
			if (browser) {
				window.removeEventListener('devicemotion', handleMotion);
			}
		};
	});
</script>

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
		background: linear-gradient(135deg, #6e8efb, #88d3ce);
		font-family: 'Arial', sans-serif;
		user-select: none;
	}

	h1 {
		color: white;
		font-size: 2.5rem;
		margin-bottom: 1rem;
		text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
	}

	.number {
		font-size: 5rem;
		font-weight: bold;
		color: white;
		background: rgba(0, 0, 0, 0.2);
		border-radius: 1rem;
		padding: 1rem 2rem;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.2);
		transition: transform 0.2s ease;
	}

	.number.shake {
		animation: shake 0.3s ease;
	}

	@keyframes shake {
		0% { transform: translateX(0); }
		25% { transform: translateX(-10px); }
		50% { transform: translateX(10px); }
		75% { transform: translateX(-10px); }
		100% { transform: translateX(0); }
	}

	.instructions {
		color: white;
		font-size: 1.2rem;
		margin-top: 1rem;
		opacity: 0.9;
	}
</style>

<div class="container">
	<h1>Shake Dice</h1>
	<div class="number" class:shake={number !== '-' && lastShake > 0}>{number}</div>
	<p class="instructions">Shake your phone to roll!</p>
</div>