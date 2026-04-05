<script lang="ts">
	let {
		onclick,
		pos = $bindable(),
		pinPos = $bindable()
	} = $props<{
		onclick: (e: MouseEvent) => void;
		pos: { x: number; y: number };
		pinPos: { x: number; y: number };
	}>();

	let pin: HTMLElement;
	$effect(() => {
		const _ = pos;
		const rect = pin.getBoundingClientRect();
		pinPos = { x: rect.left, y: rect.top };
	});
</script>

<div bind:this={pin} {onclick} class="absolute -top-5 left-1/2 h-10 w-10">
	<img
		src="/src/lib/assets/pushpin-down.png"
		class="z-[999] mx-auto h-auto h-full w-10 w-full cursor-pointer select-none"
	/>
</div>
