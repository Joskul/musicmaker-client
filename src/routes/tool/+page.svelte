<script>
	import Layout from '../+layout.svelte';
	import Dropzone from '../../components/Dropzone.svelte';
</script>

<Layout class="flex flex-col justify-center items-center">
	<p class="text-5xl m-8">Split tracks into <b>MIDI</b> files✨</p>
	<Dropzone class="rounded-lg m-8" />
</Layout>
