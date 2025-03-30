<script lang="ts">
	import { load } from "@tauri-apps/plugin-store";
	import type { Store } from "@tauri-apps/plugin-store";

	let canSmoking = true;
	let addTime = 1000 * 60 * 10; // 10 minutes
	let startingInterval = 1000 * 60 * 60; // 1 hour
	let remainingTime = startingInterval; // Remaining time in milliseconds
	let endTime: number | null = null; // Timestamp for countdown end
	let timer: ReturnType<typeof setInterval> | null = null;

	// Load the store
	let store: Store;

	const smokingNow = async () => {
		canSmoking = false;
		await store.set("canSmoking", canSmoking); // Save to store

		// Set endTime and save it
		endTime = Date.now() + remainingTime;
		await store.set("endTime", endTime);

		startCountdown();
	};

	const startCountdown = () => {
		if (timer) clearInterval(timer); // Clear any existing timer
		timer = setInterval(async () => {
			const now = Date.now();
			remainingTime = Math.max(0, (endTime ?? 0) - now); // Calculate remaining time
			await store.set("remainingTime", remainingTime); // Save to store

			if (remainingTime <= 0) {
				clearInterval(timer as unknown as number);
				timer = null;
				endOfCountdown();
			}
		}, 1000);
	};

	const endOfCountdown = async () => {
		canSmoking = true;
		await store.set("canSmoking", canSmoking); // Save to store
		startingInterval += addTime; // Increase the interval
		await store.set("startingInterval", startingInterval); // Save to store
		remainingTime = startingInterval;
		await store.set("remainingTime", remainingTime); // Save to store
		endTime = null;
		await store.set("endTime", endTime); // Clear endTime
	};

	// Reset store and variables
	const resetStore = async () => {
		canSmoking = true;
		startingInterval = 1000 * 60 * 60; // Reset to 1 hour
		remainingTime = startingInterval;
		endTime = null;

		// Clear store values
		await store.set("canSmoking", canSmoking);
		await store.set("startingInterval", startingInterval);
		await store.set("remainingTime", remainingTime);
		await store.set("endTime", endTime);

		// Clear any active timer
		if (timer) {
			clearInterval(timer);
			timer = null;
		}
	};

	(async () => {
		store = await load("store.json", { autoSave: true });

		// Load persistent values
		canSmoking = ((await store.get("canSmoking")) as boolean) ?? true;
		startingInterval = ((await store.get("startingInterval")) as number) ?? 1000 * 60 * 60;
		remainingTime = ((await store.get("remainingTime")) as number) ?? startingInterval;
		endTime = ((await store.get("endTime")) as number | null) ?? null;

		// If there's an endTime, calculate remaining time
		if (endTime) {
			const now = Date.now();
			remainingTime = Math.max(0, endTime - now);
			if (remainingTime > 0) {
				canSmoking = false;
				startCountdown();
			} else {
				endOfCountdown();
			}
		}
	})();
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

	<!-- Reset Button -->
	<div class="mt-5 text-center">
		<button on:click={resetStore} class="btn btn-xs">RESET</button>
	</div>
</div>
