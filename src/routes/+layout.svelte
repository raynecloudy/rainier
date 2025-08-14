<script lang="ts">
	import favicon from "$lib/assets/favicon.svg";

	let { children } = $props();
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<script src="https://kit.fontawesome.com/b7ef319e31.js" crossorigin="anonymous"></script>
</svelte:head>

{@render children?.()}
