<script lang="ts">
	let canvas: HTMLCanvasElement;
	let isDrawing = false;
	let startX: number, startY: number;
	let color = '#000000';
	let canvasRect: DOMRect;

	function handleMouseDown(event: MouseEvent) {
		isDrawing = true;
		canvasRect = canvas.getBoundingClientRect();
		startX = event.clientX - canvasRect.left;
		startY = event.clientY - canvasRect.top;
	}

	function handleMouseMove(event: MouseEvent) {
		if (!isDrawing) return;
		const ctx = canvas.getContext('2d');

		if (!ctx) return;

		const x = event.clientX - canvasRect.left;
		const y = event.clientY - canvasRect.top;

		ctx.strokeStyle = color;
		ctx.lineJoin = 'round';
		ctx.lineCap = 'round';
		ctx.lineWidth = 5;

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

	function clearCanvas() {
		const ctx = canvas.getContext('2d');
		if (!ctx) return;
		ctx.clearRect(0, 0, canvas.width, canvas.height);
	}
</script>

<canvas
	bind:this={canvas}
	width="600"
	height="400"
	on:mousedown={handleMouseDown}
	on:mousemove={handleMouseMove}
	on:mouseup={handleMouseUp}
/>
<div>
	<input type="color" bind:value={color} />
	<button on:click={clearCanvas}>Clear</button>
</div>

<style>
	canvas {
		border: 1px solid #000;
	}
</style>
