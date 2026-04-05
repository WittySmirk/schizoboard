<script lang="ts" module>
	export type entity = {
		type: 'note' | 'picture' | 'document';
		src?: string;
		initial?: { x: number; y: number };
	};
</script>

<script lang="ts">
	import Picture from '$lib/components/picture.svelte';
	import Note from '$lib/components/note.svelte';
	import Document from '$lib/components/document.svelte';
	import Toolbar from '$lib/components/toolbar.svelte';

	let entities: entity[] = $state([]);
	let pinpos: { x: number; y: number }[] = $state([]);

	let connections: { i1: number; i2: number }[] = $state([]);

	let pos: { x: number; y: number } = $state({ x: 0, y: 0 });
	let focused: number | undefined = $state();
	let potentialCon: number | undefined = $state();

	let zoom = $state(1);
	const ZOOM_FACTOR = 0.05;
	let scrollVal = $state(0);
	let isPanning = false;
	let lastPan = $state({ x: 0, y: 0 });
	let offset = $state({ x: 0, y: 0 });
	let drop: { x: number; y: number } | undefined = $state();

	let create = $state<'note' | 'picture' | 'document' | undefined>(undefined);

	let zCounter = $state(1);
	let selectedYarn: number | undefined;

	function parseFiles(files: FileList, initial?: { x: number; y: number }) {
		[...files].map((file) => {
			console.log(file.type);
			if (file.type.includes('image')) {
				const src = URL.createObjectURL(file);
				entities.push({ type: 'picture', src, initial });
				pinpos.push(initial ?? { x: 0, y: 0 });
			}
			if (file.type.includes('pdf')) {
				const src = URL.createObjectURL(file);
				entities.push({ type: 'document', src, initial });
				pinpos.push(initial ?? { x: 0, y: 0 });
			}
		});
	}

	function ondragover(e: Event) {
		e.preventDefault();
	}

	function ondrop(e: DragEvent) {
		e.preventDefault();
		if (e.dataTransfer != null && e.dataTransfer.files) {
			//pos = { x: (e.clientX - offset.x) / zoom, y: (e.clientY - offset.y) / zoom };
			drop = { x: (e.clientX - offset.x) / zoom, y: (e.clientY - offset.y) / zoom };
			const files = e.dataTransfer.files;
			parseFiles(files);
		}
	}

	$effect(() => {
		if (create != undefined) {
			switch (create) {
				case 'note':
					const initial = { x: (pos.x - offset.x) / zoom, y: (pos.y - offset.y) / zoom };

					entities.push({
						type: 'note',
						initial
					});
					pinpos.push(initial);

					break;
				case 'picture':
					{
						const input: HTMLInputElement = document.createElement('input');
						input.type = 'file';
						input.accept = 'image/*';
						input.onchange = (ev: Event) => {
							const files = (ev.currentTarget as HTMLInputElement).files;
							if (files != null) {
								parseFiles(files, {
									x: (pos.x - offset.x) / zoom,
									y: (pos.y - offset.y) / zoom
								});
							}
						};
						input.click();
					}
					break;
				case 'document':
					{
						const input: HTMLInputElement = document.createElement('input');
						input.type = 'file';
						input.accept = 'application/pdf';
						input.onchange = (ev: Event) => {
							const files = (ev.currentTarget as HTMLInputElement).files;
							if (files != null) {
								parseFiles(files, {
									x: (pos.x - offset.x) / zoom,
									y: (pos.y - offset.y) / zoom
								});
							}
						};
						input.click();
					}
					break;
			}
			create = undefined;
		}
		focused = undefined;
		potentialCon = undefined;
		selectedYarn = undefined;
	});

	function onmousemove(e: MouseEvent) {
		e.preventDefault();
		pos = { x: e.clientX, y: e.clientY };

		if (!isPanning) return;

		const dx = pos.x - lastPan.x;
		const dy = pos.y - lastPan.y;

		offset.x += dx;
		offset.y += dy;

		lastPan = { x: e.clientX, y: e.clientY };
		/*
		console.log(offset.x);
		console.log(offset.y);
		*/
	}

	function onwheel(e: WheelEvent) {
		e.preventDefault();
		let prevZoom = zoom;
		let newZoom = zoom + (e.deltaY > 0 ? -ZOOM_FACTOR : ZOOM_FACTOR);

		if (newZoom < 0.1 || newZoom > 2) return;
		zoom = newZoom;

		offset.x = e.clientX - (e.clientX - offset.x) * (zoom / prevZoom);
		offset.y = e.clientY - (e.clientY - offset.y) * (zoom / prevZoom);
	}

	function onmousedown(e: MouseEvent) {
		isPanning = true;
		lastPan = { x: e.clientX, y: e.clientY };
	}

	function onmouseup() {
		isPanning = false;
	}

	function onkeydown(e: KeyboardEvent) {
		console.log('here', focused);
		if (e.key == 'Delete') {
			if (focused != undefined && selectedYarn == undefined) {
				console.log('removing at ', focused);
				entities.splice(focused, 1);
				pinpos.splice(focused, 1);
				connections.map((c, i) => {
					if (c.i1 == focused || c.i2 == focused) {
						connections.splice(i, 1);
						connections = connections;
					}
				});
				entities = entities;
				pinpos = pinpos;
				focused = undefined;
			}
			if (selectedYarn != undefined) {
				console.log(selectedYarn);
				connections.splice(selectedYarn, 1);
				connections = connections;
				selectedYarn = undefined;
			}
		}
	}

	function createconn(index: number) {
		console.log('help me', index);
		if (potentialCon == undefined) {
			potentialCon = index;
			return;
		}

		connections.push({ i1: potentialCon, i2: index });
		connections = connections;

		potentialCon = undefined;
	}
</script>

<head>
	<title>schizoboard</title>
</head>

<svelte:window {onwheel} bind:scrollY={scrollVal} {onmousemove} {onkeydown} />

<div
	class="fixed inset-0"
	style="transform-origin: 0 0; transform: translate({offset.x}px, {offset.y}px) scale({zoom})"
>
	<canvas
		class="fixed -z-9999 h-full w-full overflow-hidden bg-[url(/src/lib/assets/corkboard.jpg)] bg-size-[200px] bg-repeat inset-shadow-[0_0_200px_rgba(0,0,0,0.9)] brightness-95"
		style="transform-origin: 0 0; transform: scale({1 /
			zoom}) translate({-offset.x}px, {-offset.y}px);
					background-size: {50 * zoom}%;
					background-position: {offset.x}px {offset.y}px;"
		{ondragover}
		{ondrop}
		{onmousedown}
		{onmouseup}
	>
	</canvas>

	{#each entities as entity, i}
		{#if entity.type == 'picture'}
			<Picture
				{createconn}
				bind:focused
				bind:pos
				bind:zoom
				bind:pinPos={pinpos[i]}
				bind:zCounter
				src={entity.src}
				index={i}
				{drop}
				initialPos={entity.initial}
			/>
		{:else if entity.type == 'note'}
			<Note
				{createconn}
				bind:focused
				bind:pos
				bind:zoom
				bind:pinPos={pinpos[i]}
				bind:zCounter
				index={i}
				initialPos={entity.initial!}
			/>
		{:else if entity.type == 'document'}
			<Document
				{createconn}
				bind:focused
				bind:pos
				bind:zoom
				bind:pinPos={pinpos[i]}
				bind:zCounter
				type="pdf"
				src={entity.src!}
				index={i}
				{drop}
				initialPos={entity.initial}
			/>
		{/if}
	{/each}
</div>

{#each connections as connection, i}
	{@const p1 = { x: pinpos[connection.i1].x - 20 * zoom, y: pinpos[connection.i1].y - 15 * zoom }}
	{@const p2 = { x: pinpos[connection.i2].x - 20 * zoom, y: pinpos[connection.i2].y - 15 * zoom }}
	{@const dx = p2.x - p1.x}
	{@const dy = p2.y - p1.y}
	{@const len = Math.sqrt(dx * dx + dy * dy)}
	{@const angle = (Math.atan2(dy, dx) * 180) / Math.PI}
	{@const patternH = Math.min(Math.ceil(25 * zoom), 18)}
	{@const patternY = -Math.floor(patternH / 2)}
	{@const patternW = Math.ceil(200 * zoom)}

	<svg class="pointer-events-none fixed top-0 left-0 h-full w-full">
		<defs>
			<pattern
				id="string-{connection.i1}-{connection.i2}"
				patternUnits="userSpaceOnUse"
				x="0"
				y={patternY}
				width={patternW}
				height={patternH}
			>
				<image
					href="/src/lib/assets/redyarn.png"
					x="0"
					y="0"
					width={patternW}
					height={patternH}
					preserveAspectRatio="none"
				/>
			</pattern>
		</defs>
		<g transform="translate({p1.x}, {p1.y}) rotate({angle})">
			<rect
				onclick={() => (selectedYarn = i)}
				class="pointer-events-auto hover:cursor-pointer"
				x="0"
				y={patternY}
				width={len}
				height={patternH}
				fill="url(#string-{connection.i1}-{connection.i2})"
			/>
		</g>
	</svg>
{/each}

{#each pinpos as pin}
	<img
		src="/src/lib/assets/pushpin-down.png"
		class="background-red-400 fixed z-999 mx-auto cursor-pointer drop-shadow-[19px_6px_12px_rgba(0,0,0,0.4)] select-none"
		style="transform: translate({pin.x - 35 * zoom}px, {pin.y - 30 * zoom}px); width: {40 *
			zoom}px; height: {40 * zoom}px;"
	/>
{/each}

<Toolbar bind:create bind:zoom />
