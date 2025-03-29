<script lang="ts">
	let canSmoking = true;
	// let startingInterval = 1000 * 10; // 10 seconds
	// let addTime = 1000 * 5; // 5 seconds
	let addTime = 1000 * 60 * 10; // 10 minutes
	let startingInterval = 1000 * 60 * 60; // 1 hour
	let remainingTime = startingInterval; // Remaining time in milliseconds
	let timer: ReturnType<typeof setInterval> | null = null;

	const smokingNow = () => {
		canSmoking = false;
		startCountdown();
	};

	const startCountdown = () => {
		if (timer) clearInterval(timer); // Clear any existing timer
		timer = setInterval(() => {
			remainingTime -= 1000; // Decrease by 1 second
			if (remainingTime <= 0) {
				clearInterval(timer as unknown as number);
				timer = null;
				endOfCountdown();
			}
		}, 1000);
	};

	const endOfCountdown = () => {
		// console.log("End of time");
		canSmoking = true;
		// Notify the user that they can smoke again
		// persistent storage
		startingInterval += addTime; // Increase the interval
		remainingTime = startingInterval;
	};
</script>

<div class="p-5 text-white">
	<div
		class={`card flex flex-col justify-center gap-5 space-y-10 p-5 text-center font-mono
    ${canSmoking ? "bg-green-500" : "bg-red-500"}`}
	>
		<h1 class="text-sm">Sigara Killer</h1>
		<p class="text-8xl {!canSmoking && 'underline'}">
			{canSmoking ? "YOU CAN SMOKE" : "YOU CAN NOT SMOKE"}
		</p>
		{#if canSmoking}
			<button on:click={smokingNow} class="btn btn-warning btn-lg">I AM SMOKING NOW</button>
		{:else}
			<div class="text-sm">
				<p>Time Remaining:</p>
				{#if Math.floor(remainingTime / (1000 * 60 * 60)) > 0}
					{Math.floor(remainingTime / (1000 * 60 * 60))} hours
				{/if}
				{#if Math.floor((remainingTime % (1000 * 60 * 60)) / (1000 * 60)) > 0}
					{Math.floor((remainingTime % (1000 * 60 * 60)) / (1000 * 60))} minutes
				{/if}
				{#if Math.floor((remainingTime % (1000 * 60)) / 1000) > 0}
					{Math.floor((remainingTime % (1000 * 60)) / 1000)} seconds
				{/if}
			</div>
		{/if}
	</div>
</div>
