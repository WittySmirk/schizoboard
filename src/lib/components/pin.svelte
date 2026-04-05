<script lang="ts">
	let { onclick } = $props<{ onclick: (e: MouseEvent) => void }>();
</script>

<div {onclick} class="absolute -top-5 left-1/2 h-10 w-10">
	<img
		src="/src/lib/assets/pushpin-down.png"
		class="z-[999] mx-auto h-auto h-full w-10 w-full cursor-pointer select-none"
	/>
</div>
