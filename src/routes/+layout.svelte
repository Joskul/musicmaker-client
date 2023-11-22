<script>
	import '../app.css';
	import Drawer from '../components/Drawer.svelte';
</script>

<Drawer />
<div class="{$$props.class} h-screen w-full">
	<slot />
</div>
