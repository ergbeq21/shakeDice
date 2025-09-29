<script>
    let number = null;
    let lastShake = 0;

    function roll() {
        number = Math.floor(Math.random() * 6) + 1;
        document.getElementById("result").textContent = number;
    }

    window.addEventListener("devicemotion", (event) => {
        const acc = event.accelerationIncludingGravity;
        if (!acc) return;

        const totalAcc = Math.sqrt(
            acc.x * acc.x + acc.y * acc.y + acc.z * acc.z
        );

        const now = Date.now();

        if (totalAcc > 15) {
            if (now - lastShake > 1000) {
                console.log("shake detected");
                lastShake = now;
                roll();
            }
        }
    });
</script>

<div>
  <h1> Shake Dice </h1>
  <p id="result">-</p>
</div>
