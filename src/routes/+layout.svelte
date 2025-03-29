<script lang="ts">
	import '../app.css';

	let { children } = $props();
</script>

<svelte:head>
	<title>Sigara Killer</title>
</svelte:head>

{@render children()}
