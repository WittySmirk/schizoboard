<script lang="ts">
	import Draggable from './draggable.svelte';

	let {
		initialPos,
		src = 'https://external-preview.redd.it/so-my-friend-introduced-me-to-the-theory-that-pepe-silvia-v0-rHlx3CX3RCXzhXTxnPX5ejpEmjXQjLRJjCPheSpx7Qk.jpg?width=640&crop=smart&auto=webp&s=b469871644e6ed5307b24c501f5747e3be9cb2a4',
		pos = $bindable({ x: 0, y: 0 }),
		index,
		drop,
		focused = $bindable(),
		zoom = $bindable(1)
	} = $props<{
		initialPos?: { x: number; y: number };
		src?: string;
		pos: { x: number; y: number };
		drop?: { x: number; y: number };
		index: number;
		focused: number | undefined;
		zoom: number;
	}>();

	let aspectRatio: number = $state(0);
	let natSize: { w: number; h: number } = $state({ w: 0, h: 0 });
	let loaded = $state(false);
	function onimageload(e: Event) {
		const img = e.currentTarget as HTMLImageElement;

		aspectRatio = img.naturalWidth / img.naturalHeight;
		natSize = { w: img.naturalWidth, h: img.naturalHeight };
		loaded = true;
	}

	let down = $state(false);

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

<img {src} onload={onimageload} class="hidden" />

{#if loaded}
	<Draggable
		bind:pos
		bind:zoom
		bind:down
		initialPos={{
			x: drop ? drop.x : initialPos.x,
			y: drop ? drop.y : initialPos.y,
			w: natSize.w,
			h: natSize.h
		}}
		bind:focused
		{index}
		type="picture"
		{aspectRatio}
	>
		<img class="h-full w-full object-cover select-none drop-shadow-[19px_10px_21px_rgba(0,0,0,0.2)]" style="transform: rotate({angle}deg)" {src} />
	</Draggable>
{/if}
