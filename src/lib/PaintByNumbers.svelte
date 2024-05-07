<script lang="ts">
	import { onMount } from 'svelte';
	import { gaussianRandom } from './gaussianRandom.js';

	let baseImage: HTMLImageElement;
	let canvas_dst: HTMLCanvasElement; // Destination (painting) canvas
	let canvas_src: HTMLCanvasElement; // Source (photo) canvas
	let isDrawing = false;
	let startX: number, startY: number;
	let canvasRect: DOMRect;

	onMount(() => {
		canvas_src = document.createElement('canvas');
		canvas_src.width = baseImage.width;
		canvas_src.height = baseImage.height;
		canvas_src.getContext('2d')?.drawImage(baseImage, 0, 0, baseImage.width, baseImage.height);

		canvas_dst.width = baseImage.width;
		canvas_dst.height = baseImage.height;
	});

	function handleMouseDown(event: MouseEvent) {
		isDrawing = true;
		canvasRect = canvas_dst.getBoundingClientRect();
		startX = event.clientX - canvasRect.left;
		startY = event.clientY - canvasRect.top;

		setColor(startX, startY);
	}

	function handleMouseMove(event: MouseEvent) {
		if (!isDrawing) return;

		const ctx = canvas_dst.getContext('2d');
		if (!ctx) return;

		const x = event.clientX - canvasRect.left;
		const y = event.clientY - canvasRect.top;

		ctx.beginPath();
		ctx.moveTo(startX, startY);
		ctx.lineTo(x, y);
		ctx.stroke();

		startX = x;
		startY = y;
	}

	function handleMouseUp() {
		isDrawing = false;
	}

	function setBrush(width = 5) {
		const ctx = canvas_dst.getContext('2d');
		if (!ctx) return;

		ctx.lineJoin = 'round';
		ctx.lineCap = 'round';
		ctx.lineWidth = width;
	}

	// Queries the source canvas for the color at the given location (x,y)
	function setColor(x: number, y: number) {
		const pixelData = canvas_src.getContext('2d')?.getImageData(x, y, 1, 1).data;
		if (!pixelData) return;

		const color = `rgb(${pixelData[0]}, ${pixelData[1]}, ${pixelData[2]})`;

		const ctx = canvas_dst.getContext('2d');
		if (!ctx) return;
		ctx.strokeStyle = color;
	}

	function drawStroke(xStart: number, yStart: number, xEnd: number, yEnd: number) {
		const ctx = canvas_dst.getContext('2d');
		if (!ctx) return;

		setColor(xStart, yStart);
		setBrush(4);
		ctx.beginPath();
		ctx.moveTo(xStart, yStart);
		ctx.lineTo(xEnd, yEnd);
		ctx.stroke();
	}

	function clearCanvas() {
		const ctx = canvas_dst.getContext('2d');
		if (!ctx) return;
		ctx.clearRect(0, 0, canvas_dst.width, canvas_dst.height);
	}

	function autostroke(n = 100) {
		for (let i = 0; i < n; i++) {
			const x = gaussianRandom(baseImage.width / 2, baseImage.width * 0.2); // Starting X
			const y = gaussianRandom(baseImage.height / 2, baseImage.height * 0.2); // Starting Y
			const a = gaussianRandom(Math.PI / 2, 0.2); // Angle (radians)
			const len = gaussianRandom(2, 2);
			drawStroke(x, y, x + len * Math.cos(a), y + len * Math.sin(a));
		}
	}
</script>

<div>
	<img
		bind:this={baseImage}
		width="600"
		class="preview"
		src="ou-flag-flowers.jpg"
		alt="OU Flag with Flowers"
	/>
	<canvas
		bind:this={canvas_dst}
		class="preview"
		on:mousedown={handleMouseDown}
		on:mousemove={handleMouseMove}
		on:mouseup={handleMouseUp}
	/>
</div>
<div>
	<button on:click={clearCanvas}>Clear</button>
	<button on:click={() => autostroke(100)}>Autostroke (x100)</button>
	<button on:click={() => autostroke(1000)}>Autostroke (x1000)</button>
</div>

<style>
	.preview {
		border: 2px solid #000;
	}
</style>
