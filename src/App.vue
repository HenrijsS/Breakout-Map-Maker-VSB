<template>
	<div class="deez">
		<MapGrid />
	</div>
</template>

<script setup lang="ts">
	import MapGrid from "./components/MapGrid.vue";
</script>

<style lang="scss">
	html,
	body {
		margin: 0;
		padding: 0;
	}

	.deez {
		display: flex;
		flex-direction: column;
	}
</style>
