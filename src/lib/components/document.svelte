<script lang="ts">
	import Draggable from './draggable.svelte';
	import { PdfViewer, PdfRenderer } from 'svelte-pdf-view';

	let {
		initialPos,
		pos = $bindable({ x: 0, y: 0 }),
		src,
		index,
		drop,
		focused = $bindable(),
		zoom = $bindable(1),
		createconn,
		pinPos = $bindable(),
		zCounter = $bindable()
	} = $props<{
		initialPos?: { x: number; y: number };
		pos: { x: number; y: number };
		type: 'pdf' | 'md';
		src: string;
		index: number;
		drop?: { x: number; y: number };
		focused: number | undefined;
		zoom: number;
		createconn: (index: number) => void;
		pinPos: { x: number; y: number };
		zCounter: number;
	}>();

	let down = $state(false);

	let velX = 0;
	let smoothVelX = 0;
	let prev = { x: 0, t: performance.now() };

	const DELTA_ANGLE = 4;
	let angle = $state(0);

	$effect(() => {
		if (!down) {
			return;
		}
		const now = performance.now();

		if (!prev) {
			prev = { x: pos.x, t: now };
			return;
		}

		const dt = now - prev.t;

		if (dt > 5) {
			velX = (pos.x - prev.x) / dt;
			smoothVelX = 0.8 * smoothVelX + 0.2 * velX;
			angle = smoothVelX * DELTA_ANGLE;
		}

		prev = { x: pos.x, t: now };
	});
</script>

<Draggable
	bind:focused
	bind:pos
	bind:down
	bind:zoom
	bind:pinPos
	bind:zCounter
	{createconn}
	{index}
	initialPos={{ x: drop ? drop.x : initialPos.x, y: drop ? drop.y : initialPos.y, w: 650, h: 875 }}
	type="document"
>
	<div class="h-full select-none"   style="transform: rotate({angle}deg)">
		<PdfViewer {src}>
			<PdfRenderer backgroundColor="#FFFFFF" />
		</PdfViewer>
	</div>
</Draggable>
