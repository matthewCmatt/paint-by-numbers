<script lang="ts">
	import { onMount } from 'svelte';
	import { gaussianRandom } from './gaussianRandom.js';

	interface Props {
		imageURL: string;
		strokeRadius?: number;
		strokeLength?: number;
		strokeAngle?: number;
	}

	let { imageURL, strokeRadius = 5, strokeLength = 10, strokeAngle = 2.7 }: Props = $props();

	let baseImage: HTMLImageElement;
	let canvas_dst: HTMLCanvasElement; // Destination (painting) canvas
	let canvas_src: HTMLCanvasElement; // Source (photo) canvas
	let isDrawing = false;
	let startX: number, startY: number;
	let canvasRect: DOMRect;

	onMount(() => {
		if (baseImage.complete) {
			setupCanvas();
		} else {
			baseImage.onload = () => {
				setupCanvas();
			};
		}
	});

	function setupCanvas() {
		canvas_dst.width = baseImage.width;
		canvas_dst.height = baseImage.height;

		canvas_src = document.createElement('canvas');
		canvas_src.width = baseImage.width;
		canvas_src.height = baseImage.height;
		canvas_src.getContext('2d')?.drawImage(baseImage, 0, 0, baseImage.width, baseImage.height);
	}

	function handleMouseDown(event: MouseEvent) {
		isDrawing = true;
		canvasRect = canvas_dst.getBoundingClientRect();
		startX = event.clientX - canvasRect.left;
		startY = event.clientY - canvasRect.top;

		setColor(startX, startY);
		setBrush();
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

	function setBrush() {
		const ctx = canvas_dst.getContext('2d');
		if (!ctx) return;

		ctx.lineJoin = 'round';
		ctx.lineCap = 'round';
		ctx.lineWidth = strokeRadius;
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
		setBrush();
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
			const a = gaussianRandom(strokeAngle, 0.2); // Angle (radians)
			const length = gaussianRandom(strokeLength, 2);
			drawStroke(x, y, x + length * Math.cos(a), y + length * Math.sin(a));
		}
	}
</script>

<div class="root">
	<!-- svelte-ignore a11y_missing_attribute -->
	<img src={imageURL} class="preview" width="600" bind:this={baseImage} />
	<canvas
		class="preview"
		width="600"
		bind:this={canvas_dst}
		onmousedown={handleMouseDown}
		onmousemove={handleMouseMove}
		onmouseup={handleMouseUp}
	></canvas>
	<div>
		<button onclick={clearCanvas}>Clear</button>
		<button onclick={() => autostroke(100)}>Autostroke (x100)</button>
		<button onclick={() => autostroke(1000)}>Autostroke (x1000)</button>
	</div>
</div>

<style>
	.root {
		text-align: center;
	}
	.preview {
		border: 1px solid #000;
		border-radius: 5px;
	}
</style>
