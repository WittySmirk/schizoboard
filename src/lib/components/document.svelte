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
		createconn
	} = $props<{
		initialPos?: { x: number; y: number };
		pos: { x: number; y: number };
		type: 'pdf' | 'md';
		src: string;
		index: number;
		drop?: { x: number; y: number };
		focused: number | undefined;
		zoom: number;
		createconn: (pos: { x: number; y: number }) => void;
	}>();

	let down = $state(false);
</script>

<Draggable
	bind:focused
	bind:pos
	bind:down
	bind:zoom
	{createconn}
	{index}
	initialPos={{ x: drop ? drop.x : initialPos.x, y: drop ? drop.y : initialPos.y, w: 650, h: 875 }}
	type="document"
>
	<div class="h-full select-none">
		<PdfViewer {src}>
			<PdfRenderer backgroundColor="#FFFFFF" />
		</PdfViewer>
	</div>
</Draggable>
