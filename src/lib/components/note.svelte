<script lang="ts">
	import Draggable from './draggable.svelte';

	let {
		initialPos,
		pos = $bindable({ x: 0, y: 0 }),
		index,
		zoom = $bindable(1),
		focused = $bindable()
	} = $props<{
		initialPos: { x: number; y: number };
		pos: { x: number; y: number };
		index: number;
		focused: number | undefined;
		zoom: number;
	}>();
	let text = $state('Test');

	let down = $state(false);
	let doubleclick = $state(false);

	function onblur() {
		doubleclick = false;
	}

	let velX = 0;
	let smoothVelX = 0;
	let prev = { x: 0, t: performance.now() };

	const DELTA_ANGLE = 5;
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
	initialPos={{ x: initialPos.x, y: initialPos.y, w: 200, h: 200 }}
	bind:pos
	bind:down
	bind:doubleclick
	bind:zoom
	bind:focused
	{index}
	type="note"
	aspectRatio={1}
>
	<div
		role="navigation"
		style="transform: rotate({angle}deg)"
		class="h-full w-full bg-[url(/src/lib/assets/stickynote.png)] bg-cover select-none"
	>
		{#if !doubleclick}
			<p class="absolute inset-3 resize-none overflow-hidden wrap-break-word whitespace-pre-wrap select-none">
				{text}
			</p>
		{:else}
			<textarea
				class="absolute inset-3 resize-none overflow-hidden wrap-break-word whitespace-pre-wrap"
				bind:value={text}
				autofocus
				{onblur}
			>
			</textarea>
			<p></p>
		{/if}
	</div>
</Draggable>
