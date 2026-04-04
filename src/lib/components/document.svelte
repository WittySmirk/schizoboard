<script lang="ts">
	import Draggable from './draggable.svelte';
	import { PdfViewer, PdfRenderer } from 'svelte-pdf-view';

	let {
		pos = $bindable({ x: 0, y: 0 }),
		type,
		src,
		index,
		dragged = false,
		focused = $bindable()
	} = $props<{
		pos: { x: number; y: number };
		type: 'pdf' | 'md';
		src: string;
		index: number;
		dragged: boolean;
		focused: number | undefined;
	}>();

	let down = $state(false);
</script>

<Draggable
	bind:focused
	bind:pos
	bind:down
	{index}
	initialX={dragged ? pos.x : 0}
	initialY={dragged ? pos.y : 0}
>
	<div class="h-205">
		<PdfViewer {src}>
			<PdfRenderer backgroundColor="#FFFFFF" />
		</PdfViewer>
	</div>
</Draggable>
