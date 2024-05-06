<script lang="ts">
	let isDrawing = false;
	let startX: number, startY: number;
	let color = '#000000';

	function handleMouseDown(event: MouseEvent) {
		isDrawing = true;
		startX = event.clientX;
		startY = event.clientY;
	}

	function handleMouseMove(event: MouseEvent) {
		if (!isDrawing) return;

		const canvas = document.getElementById('canvas') as HTMLCanvasElement;
		const ctx = canvas.getContext('2d');

		if (!ctx) return;

		const x = event.clientX - canvas.offsetLeft;
		const y = event.clientY - canvas.offsetTop;

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
		const canvas = document.getElementById('canvas') as HTMLCanvasElement;
		const ctx = canvas.getContext('2d');
		if (!ctx) return;
		ctx.clearRect(0, 0, canvas.width, canvas.height);
	}
</script>

<div>
	<canvas
		id="canvas"
		width="600"
		height="400"
		on:mousedown={handleMouseDown}
		on:mousemove={handleMouseMove}
		on:mouseup={handleMouseUp}
	></canvas>
	<div>
		<input type="color" bind:value={color} />
		<button on:click={clearCanvas}>Clear</button>
	</div>
</div>

<style>
	#canvas {
		border: 1px solid #000;
	}
</style>
