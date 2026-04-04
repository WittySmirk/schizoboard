<script lang="ts">
	import Draggable from "./draggable.svelte";
	import { PdfViewer, PdfRenderer } from "svelte-pdf-view";

	let {
		pos = $bindable({ x: 0, y: 0 }),
		type,
		src,
		zoom = $bindable(1),
	} = $props<{
		pos: { x: number; y: number };
		type: "pdf" | "md";
		src: string;
		zoom: number;
	}>();
	let down = $state(false);
</script>

<Draggable bind:pos bind:down bind:zoom>
	<PdfViewer {src}>
		<PdfRenderer backgroundColor="#e8e8e8" />
	</PdfViewer>
</Draggable>
