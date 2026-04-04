<script lang="ts">
	import Draggable from "./draggable.svelte";

	let { pos = $bindable({ x: 0, y: 0 }) } = $props<{
		pos: { x: number; y: number };
	}>();
	let text = $state("Test");

	let down = $state(false);
	let doubleclick = $state(false);

	function onblur() {
		doubleclick = false;
	}

	let velX = 0;
	let smoothVelX = 0;
	let prev = {x: 0, t: performance.now()};

	const DELTA_ANGLE = 5;
	let angle = $state(0);

	function onmousemove(e: MouseEvent) {
		if (!down) {
			return;
		}
		const now = performance.now();

		if (!prev) {
			prev = {x: e.clientX, t: now};
			return;
		}

		const dt = now - prev.t;

		if (dt > 5) {
			velX = (e.clientX - prev.x) / dt
			smoothVelX = (0.8 * smoothVelX) + (0.2 * velX);
			angle = smoothVelX * DELTA_ANGLE;
		}

		prev = { x: e.clientX, t: now };
    }

</script>
	<Draggable bind:pos bind:down bind:doubleclick>
		<div {onmousemove} role="navigation"   style="transform: rotate({angle}deg)"   class="relative w-48 h-48 bg-[url(/src/lib/assets/stickynote.png)]">
			{#if !doubleclick}
				<p class="absolute inset-3 overflow-hidden whitespace-pre-wrap break-words resize-none">{text}</p>
			{:else}
				<textarea class="absolute inset-3 overflow-hidden whitespace-pre-wrap break-words resize-none"
					bind:value={text}  
					autofocus 
					{onblur}>
				</textarea>
				<p></p>
			{/if}
		</div>
	</Draggable>
