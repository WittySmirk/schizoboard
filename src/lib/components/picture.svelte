<script lang="ts">
	import Draggable from './draggable.svelte';

	let {
		initialPos,
		src = 'https://external-preview.redd.it/so-my-friend-introduced-me-to-the-theory-that-pepe-silvia-v0-rHlx3CX3RCXzhXTxnPX5ejpEmjXQjLRJjCPheSpx7Qk.jpg?width=640&crop=smart&auto=webp&s=b469871644e6ed5307b24c501f5747e3be9cb2a4',
		pos = $bindable({ x: 0, y: 0 }),
		index,
		drop,
		focused = $bindable(),
		zoom = $bindable(1),
		createconn,
		pinPos = $bindable(),
		zCounter = $bindable(),
	} = $props<{
		initialPos?: { x: number; y: number };
		src?: string;
		pos: { x: number; y: number };
		drop?: { x: number; y: number };
		index: number;
		focused: number | undefined;
		zoom: number;
		createconn: (index: number) => void;
		pinPos: { x: number; y: number };
		zCounter: number;
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

{#if !loaded}
	<img {src} onload={onimageload} class="hidden" />
{/if}

{#if loaded}
	<Draggable
		{createconn}
		bind:pos
		bind:zoom
		bind:pinPos
		bind:down
		bind:zCounter
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
		<div class="h-full w-full bg-[#ffffed] brightness-95 p-6 pb-20 overflow-hidden drop-shadow-[19px_10px_21px_rgba(0,0,0,0.3)] select-none" style="transform: rotate({angle}deg)">
			<div class="relative inline block">
				<img class="w-full h-full object-cover select-none brightness-95 saturate-80" {src}/>
				
				<div class="pointer-events-none absolute inset-0 opacity-80 bg-cover mix-blend-overlay bg-[url('/images/film-grain.jpg')]"></div>
				<div class="pointer-events-none absolute inset-0 opacity-30 bg-cover mix-blend-overlay bg-[url('/images/polaroid.png')]"></div>
			</div>
		</div>
		<!-- <img class="h-full w-full object-cover select-none drop-shadow-[19px_10px_21px_rgba(0,0,0,0.2)]" style="transform: rotate({angle}deg)" {src} /> -->
	</Draggable>
{/if}
