<script lang="ts">
	let { children, perspective } = $props();
</script>

<div style:width="100%" style:height="100%" style:position="absolute" style:transform-style="preserve-3d"
    style:transform="translateZ({perspective}px)">
    {@render children()}
</div>