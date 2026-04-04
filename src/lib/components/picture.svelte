<script lang="ts">
	import Draggable from './draggable.svelte';

	let {
		src = 'https://external-preview.redd.it/so-my-friend-introduced-me-to-the-theory-that-pepe-silvia-v0-rHlx3CX3RCXzhXTxnPX5ejpEmjXQjLRJjCPheSpx7Qk.jpg?width=640&crop=smart&auto=webp&s=b469871644e6ed5307b24c501f5747e3be9cb2a4',
		pos = $bindable({ x: 0, y: 0 }),
		dragged = false,
		index,
		focused = $bindable(),
		zoom = $bindable(1)
	} = $props<{
		src?: string;
		pos: { x: number; y: number };
		dragged?: boolean;
		index: number;
		focused: number | undefined;
		zoom: number;
	}>();
</script>

<Draggable
	bind:pos
	bind:zoom
	initialX={dragged ? pos.x : 0}
	initialY={dragged ? pos.y : 0}
	bind:focused
	{index}
>
	<img {src} />
</Draggable>
