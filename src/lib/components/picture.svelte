<script lang="ts">
	import Draggable from './draggable.svelte';

	let {
		src = 'https://external-preview.redd.it/so-my-friend-introduced-me-to-the-theory-that-pepe-silvia-v0-rHlx3CX3RCXzhXTxnPX5ejpEmjXQjLRJjCPheSpx7Qk.jpg?width=640&crop=smart&auto=webp&s=b469871644e6ed5307b24c501f5747e3be9cb2a4'
	} = $props();
</script>

<Draggable>
	<img {src} />
</Draggable>
