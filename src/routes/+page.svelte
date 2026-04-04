<script lang="ts">
	import Draggable from "$lib/components/draggable.svelte";
	import Picture from "$lib/components/picture.svelte";
	import Note from "$lib/components/note.svelte";
	import Document from "$lib/components/document.svelte";

	// TODO:  make one array with different types embedded
	let pictures: string[] = $state([]);
	let documents: string[] = $state([]);
	let pos: { x: number; y: number } = $state({ x: 0, y: 0 });

	let zoom = $state(1);
	const ZOOM_FACTOR = 0.1
	let scrollVal = $state(0);
	let isPanning = false;
	let lastPan = $state({x: 0, y: 0});
	let offset = $state({x: 0, y: 0});

	function parseFiles(files: FileList) {
		[...files].map((file) => {
			console.log(file.type);
			if (file.type.includes("image")) {
				const src = URL.createObjectURL(file);
				pictures.push(src);
			}
			if (file.type.includes("pdf")) {
				const src = URL.createObjectURL(file);
				documents.push(src);
			}
		});
	}

	function onchange(e: Event & { currentTarget: HTMLInputElement }) {
		const files = e.currentTarget.files;
		if (files != null) {
			parseFiles(files);
		}
	}
	function ondragover(e: Event) {
		e.preventDefault();
	}

	function ondrop(e: DragEvent) {
		e.preventDefault();
		if (e.dataTransfer != null && e.dataTransfer.files) {
			pos = { x: e.x, y: e.y };
			const files = e.dataTransfer.files;
			parseFiles(files);
		}
	}

	function onmousemove(e: MouseEvent) {
		e.preventDefault();
		pos = {x: e.clientX, y: e.clientY};

		if (!isPanning) return;

		const dx = pos.x - lastPan.x;
		const dy = pos.y - lastPan.y;

		offset.x += dx;
		offset.y += dy;

		lastPan = {x: e.clientX, y: e.clientY};
		console.log(offset.x);
		console.log(offset.y);
	}

	function onwheel(e: WheelEvent) {
		zoom += e.deltaY > 0 ? -ZOOM_FACTOR : ZOOM_FACTOR;
	}

	function onmousedown(e: MouseEvent) {
		isPanning = true;
		lastPan = {x: e.clientX, y: e.clientY};
	}

	function onmouseup() {
		isPanning = false;
	}
</script>

<head>
	<title>schizoboard</title>
</head>


<svelte:window {onwheel} bind:scrollY={scrollVal} {onmousemove} />

<div class="fixed inset-0" style="transform: translate({offset.x}px, {offset.y}px) scale({zoom})">
	<canvas class="fixed h-full w-full bg-[url(/src/lib/assets/corkboard.jpg)] bg-size-[200px] bg-repeat inset-shadow-[0_0_200px_rgba(0,0,0,0.9)] -z-9999 brightness-95 " 
			style="transform: scale({1/zoom}) translate({-offset.x}px, {-offset.y}px);
					background-size: {50 * zoom}%;
					background-position: {offset.x}px {offset.y}px;"   
			{ondragover} {ondrop} {onmousedown} {onmouseup}>
	</canvas>

		<Draggable bind:pos bind:zoom>
			<h1>Hello diddy</h1>
		</Draggable>

		<Picture bind:pos bind:zoom></Picture>

		{#if pictures.length != 0}
			{#each pictures as picture}
				<Picture bind:pos bind:zoom src={picture} dragged={true}></Picture>
			{/each}
		{/if}

		<Note bind:pos bind:zoom></Note>

		{#if documents.length != 0}
			{#each documents as document}
				<Document bind:pos bind:zoom type="pdf" src={document}></Document>
			{/each}
		{/if}

		<!-- TODO: add styling to upload button -->
		<input type="file" id="upload" {onchange} hidden />
		<label for="upload">Upload File</label>
</div>
