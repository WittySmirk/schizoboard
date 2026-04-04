<script lang="ts">
	import Draggable from './draggable.svelte';
	import { PdfViewer, PdfRenderer } from 'svelte-pdf-view';

	let {
		pos = $bindable({ x: 0, y: 0 }),
		type,
		src,
		index,
		drop,
		focused = $bindable(),
		zoom = $bindable(1)
	} = $props<{
		pos: { x: number; y: number };
		type: 'pdf' | 'md';
		src: string;
		index: number;
		drop?: { x: number; y: number };
		focused: number | undefined;
		zoom: number;
	}>();

	let down = $state(false);
</script>

<Draggable
	bind:focused
	bind:pos
	bind:down
	bind:zoom
	{index}
	initialX={drop ? drop.x : 0}
	initialY={drop ? drop.y : 0}
>
	<div class="h-205">
		<PdfViewer {src}>
			<PdfRenderer backgroundColor="#FFFFFF" />
		</PdfViewer>
	</div>
</Draggable>
