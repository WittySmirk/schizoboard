<script lang="ts">
	import Draggable from './draggable.svelte';

	let {
		pos = $bindable({ x: 0, y: 0 }),
		index,
		zoom = $bindable(1),
		focused = $bindable()
	} = $props<{
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
	<Draggable bind:pos bind:down bind:doubleclick bind:zoom bind:focused {index}>
		<div role="navigation"   style="transform: rotate({angle}deg)"   class="relative w-48 h-48 bg-[url(/src/lib/assets/stickynote.png)]">
			{#if !doubleclick}
				<p class="absolute inset-3 overflow-hidden whitespace-pre-wrap wrap-break-word resize-none">{text}</p>
			{:else}
				<textarea class="absolute inset-3 overflow-hidden whitespace-pre-wrap wrap-break-word resize-none"
					bind:value={text}  
					autofocus 
					{onblur}>
				</textarea>
				<p></p>
			{/if}
		</div>
	</Draggable>
