<script>
	import { onMount, onDestroy } from 'svelte';
	import { browser } from '$app/environment';

	let number = '-';
	let lastShake = 0;
	let errorMessage = '';
	const SHAKE_THRESHOLD = 15; // Acceleration threshold for shake detection
	const SHAKE_COOLDOWN = 1000; // Minimum time between shakes in ms

	function roll() {
		number = Math.floor(Math.random() * 6) + 1;
	}

	function handleMotion(event) {
		if (!event.accelerationIncludingGravity) {
			errorMessage = 'No acceleration data available';
			return;
		}

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
			errorMessage = ''; // Clear any previous error
		}
	}

	function requestPermission() {
		if (typeof DeviceMotionEvent !== 'undefined' && typeof DeviceMotionEvent.requestPermission === 'function') {
			DeviceMotionEvent.requestPermission()
				.then(permissionState => {
					if (permissionState === 'granted') {
						errorMessage = '';
						window.addEventListener('devicemotion', handleMotion);
					} else {
						errorMessage = 'Motion permission denied';
					}
				})
				.catch(err => {
					errorMessage = `Permission request failed: ${err.message}`;
				});
		} else {
			// For browsers that don't require explicit permission
			window.addEventListener('devicemotion', handleMotion);
		}
	}

	onMount(() => {
		if (browser) {
			// Check if DeviceMotionEvent is supported
			if (typeof DeviceMotionEvent === 'undefined') {
				errorMessage = 'Device motion not supported by this browser or device';
				return;
			}
			requestPermission();
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

	.error {
		color: #ff4d4d;
		font-size: 1rem;
		margin-top: 1rem;
		text-align: center;
		max-width: 80%;
	}

	.fallback-button {
		margin-top: 1rem;
		padding: 0.8rem 1.5rem;
		font-size: 1.2rem;
		color: white;
		background: #4a4a4a;
		border: none;
		border-radius: 0.5rem;
		cursor: pointer;
		transition: background 0.2s ease;
	}

	.fallback-button:hover {
		background: #5a5a5a;
	}
</style>

<div class="container">
	<h1>Shake Dice</h1>
	<div class="number" class:shake={number !== '-' && lastShake > 0}>{number}</div>
	<p class="instructions">Shake your phone to roll!</p>
	{#if errorMessage}
		<p class="error">{errorMessage}</p>
	{/if}
	<button class="fallback-button" on:click={roll}>Roll Manually</button>
</div>