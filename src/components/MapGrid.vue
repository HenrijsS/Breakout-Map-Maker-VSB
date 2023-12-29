<template>
	<div class="map-grid-wrapper">
		<div class="desc">
			<span
				v-on:click="mode = 'Empty'"
				v-bind:class="{ selected: mode == 'Empty' }"
			>
				Empty
			</span>
			<span
				v-on:click="mode = 'Block'"
				v-bind:class="{ selected: mode == 'Block' }"
			>
				Block
			</span>
			<span
				v-on:click="mode = 'Wall'"
				v-bind:class="{ selected: mode == 'Wall' }"
			>
				Wall
			</span>
			<span
				v-on:click="mode = 'PowerUp'"
				v-bind:class="{ selected: mode == 'PowerUp' }"
			>
				PowerUp
			</span>
		</div>
		<div class="map-grid">
			<div
				v-for="(row, rowIndex) in grid"
				:key="rowIndex"
				class="map-grid-row"
			>
				<div
					v-for="(block, blockIndex) in row"
					:key="blockIndex"
					class="map-grid-block"
					v-on:click="toggleBlock(block)"
					v-on:mousedown="mouseDown = true"
					v-on:mouseup="mouseDown = false"
					v-on:mousemove="mouseDown && toggleBlock(block)"
				>
					<div
						v-if="block.isEmpty"
						class="map-grid-block-empty"
					></div>
					<div v-if="block.isBlock" class="map-grid-block-block">
						B
					</div>
					<div v-if="block.isWall" class="map-grid-block-wall">W</div>
					<div
						v-if="block.hasPowerUp"
						class="map-grid-block-power-up"
					>
						P
					</div>
				</div>
			</div>

			<div class="buttons">
				<button v-on:click="saveMap()">Save Map</button>
				<button v-on:click="loadMap()">Load Map</button>
			</div>

			<div class="buttons">
				<button v-on:click="restart()">Restart</button>
				<button v-on:click="emptyMap()">Empty Map</button>
			</div>
		</div>
	</div>
</template>

<script setup lang="ts">
	import { ref, reactive } from "vue";

	// Define a block type (Types: Empty, Block, PowerUp, Wall)
	type block = {
		isEmpty?: boolean;
		isBlock?: boolean;
		isWall?: boolean;
		hasPowerUp?: boolean;
	};

	const mode = ref<"Empty" | "Block" | "Wall" | "PowerUp">("Empty");
	const mouseDown = ref(false); // Add this line

	// Create a 20x16 grid of blocks
	const grid = reactive(
		Array.from({ length: 16 }, () =>
			Array.from({ length: 20 }, () => ({
				isEmpty: false,
				isBlock: true,
				isWall: false,
				hasPowerUp: false,
			}))
		)
	);

	const toggleBlock = (block: block) => {
		// Toggle the block type based on the current mode
		switch (mode.value) {
			case "Empty":
				block.isEmpty = true;
				block.isBlock = false;
				block.isWall = false;
				block.hasPowerUp = false;
				break;
			case "Block":
				block.isEmpty = false;
				block.isBlock = true;
				block.isWall = false;
				block.hasPowerUp = false;
				break;
			case "Wall":
				block.isEmpty = false;
				block.isBlock = false;
				block.isWall = true;
				block.hasPowerUp = false;
				break;
			case "PowerUp":
				block.isEmpty = false;
				block.isBlock = false;
				block.isWall = false;
				block.hasPowerUp = true;
				break;
		}
	};

	const saveMap = () => {
		// Save the map in a text file in the following format:
		// Structure: <Row>, <Col>, <IsWall>, <HasPowerup>, <IsEmpty>

		const map = grid.map((row, rowIndex) => {
			return row.map((block, blockIndex) => {
				return `${rowIndex},${blockIndex},${block.isWall ? "1" : "0"},${
					block.hasPowerUp ? "1" : "0"
				},${block.isEmpty ? "0" : "1"}`;
			});
		});

		// Add "Structure: <Row>, <Col>, <IsWall>, <HasPowerup>" at the top of the file
		map.unshift([
			"Structure: <Row>, <Col>, <IsWall>, <HasPowerup>, <IsAlive>",
		]);

		// Join the map array with a new line (Include the newline) (Remove the comma at the end of each line)
		const mapString = map
			.map((row) => row.join("\n"))
			.join("\n")
			.replace(/,\n/g, "\n");

		console.log(mapString);

		const blob = new Blob([mapString], {
			type: "text/plain;charset=utf-8",
		});

		const a = document.createElement("a");
		a.download = "map.txt";
		a.href = window.URL.createObjectURL(blob);
		a.dataset.downloadurl = ["text/plain", a.download, a.href].join(":");
		a.style.display = "none";
		document.body.appendChild(a);
		a.click();
		document.body.removeChild(a);
	};

	const loadMap = () => {
		// Load the map from a text file in the following format:
		// Structure: <Row>, <Col>, <IsWall>, <HasPowerup>

		const fileInput = document.createElement("input");
		fileInput.type = "file";
		fileInput.accept = ".txt";
		fileInput.onchange = (e) => {
			const file = (e.target as HTMLInputElement)?.files?.[0];
			const reader = new FileReader();
			reader.onload = (e) => {
				const mapString = (e.target as FileReader)?.result as string;
				const map = mapString
					.split("\n")
					.slice(1)
					.map((row) => row.split(","));
				console.log(map);

				map.forEach((blockData) => {
					const rowIndex = parseInt(blockData[0]);
					const blockIndex = parseInt(blockData[1]);
					const isWall = blockData[2] === "1";
					const hasPowerUp = blockData[3] === "1";
					const isEmpty = blockData[4] === "1" ? false : true;
					const isBlock = !isWall && !hasPowerUp && !isEmpty;

					const block = {
						isEmpty,
						isBlock,
						isWall,
						hasPowerUp,
					};

					if (!grid[rowIndex]) {
						grid[rowIndex] = [];
					}

					grid[rowIndex][blockIndex] = block;
				});
			};
			if (file) {
				reader.readAsText(file);
			}
		};
		fileInput.click();
	};

	const restart = () => {
		// Restart the map
		grid.forEach((row) => {
			row.forEach((block) => {
				block.isEmpty = false;
				block.isBlock = true;
				block.isWall = false;
				block.hasPowerUp = false;
			});
		});
	};

	const emptyMap = () => {
		// Empty the map
		grid.forEach((row) => {
			row.forEach((block) => {
				block.isEmpty = true;
				block.isBlock = false;
				block.isWall = false;
				block.hasPowerUp = false;
			});
		});
	};
</script>

<style lang="scss">
	.map-grid-wrapper {
		display: flex;
		justify-content: center;
		align-items: center;
		gap: 20px;

		.desc {
			display: flex;
			flex-direction: column;
			gap: 10px;
			font-size: 20px;
			font-weight: bold;

			span {
				padding: 5px 15px;
				border: 1px solid black;
				border-radius: 24px;
				cursor: pointer;

				&.selected {
					border: 3px solid black;
				}
			}

			span:nth-child(2) {
				color: #fff;
				background-color: blue;
			}

			span:nth-child(3) {
				color: #fff;
				background-color: gray;
			}

			span:nth-child(4) {
				color: #fff;
				background-color: red;
			}
		}
	}
	.map-grid {
		display: flex;
		flex-direction: column;
		height: 100%;
		margin: 20px 0;

		.buttons {
			display: flex;
			gap: 40px;
			justify-content: center;
		}

		button {
			width: auto;
			align-self: center;
			border-radius: 24px;
			padding: 0 20px;
			height: 50px;
			font-size: 20px;
			font-weight: bold;
			background-color: #fff;
			border: 1px solid black;
			cursor: pointer;
			margin-top: 20px;
			transition: 0.4s all;

			&:hover {
				background: black;
				color: white;
			}
		}
	}

	.map-grid-row {
		display: flex;
		flex-direction: row;
		justify-content: center;
		width: 100%;
		height: 100%;
	}

	.map-grid-block {
		width: 25px;
		height: 25px;
		background: black;
		border: 1px solid white;
		cursor: pointer;
		user-select: none;

		* {
			width: 100%;
			height: 100%;
			display: flex;
			justify-content: center;
			align-items: center;
			color: white;
			font-size: 20px;
		}
	}

	.map-grid-block-block {
		background-color: blue;
	}

	.map-grid-block-wall {
		background-color: gray;
	}

	.map-grid-block-power-up {
		background-color: red;
	}
</style>
