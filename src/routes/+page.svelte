<script>
	import { onMount, onDestroy } from "svelte";
    import { browser } from "$app/environment";

	let number = "-";
	let lastShake = 0;

	function roll() {
		number = Math.floor(Math.random() * 6) + 1;
	}

	onMount(() => {
		window.addEventListener("devicemotion", (event)=>{
            const acc = event.accelerationIncludingGravity;
            if (!acc) return;

            const totalAcc = Math.sqrt(
                (acc.x || 0) * (acc.x || 0) +
                (acc.y || 0) * (acc.y || 0) +
                (acc.z || 0) * (acc.z || 0)
            );

            const now = Date.now();

            if (totalAcc > 15 && now - lastShake > 1000) {
                console.log("shake detected");
                lastShake = now;
                roll();
            }
        });
	});

	onDestroy(() => {
		browser && window.removeEventListener("devicemotion", handleMotion);
	});
</script>

<div>
  <h1>Shake Dice</h1>
  <p>{number}</p>
</div>
