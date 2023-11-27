<script>
	import { onMount, onDestroy, afterUpdate } from 'svelte';
	import WaveSurfer from 'wavesurfer.js';

	let waveSurfer;
	let isPlaying = false;
	let progress = 0;

	export let src;

	onMount(() => {
		const container = document.getElementById('waveform');
		waveSurfer = WaveSurfer.create({
			container,
			waveColor: 'rgba(128,128,128,1)',
			progressColor: 'rgba(128,128,128,0.8)',
			responsive: true
		});

		waveSurfer.load(src);

		waveSurfer.on('audioprocess', handleAudioProcess);
	});

	onDestroy(() => {
		if (waveSurfer) {
			waveSurfer.destroy();
		}
	});

	function togglePlay() {
		if (waveSurfer) {
			isPlaying ? waveSurfer.pause() : waveSurfer.play();
			isPlaying = !isPlaying;
		}
	}

	function handleAudioProcess() {
		progress = (waveSurfer.getCurrentTime() / waveSurfer.getDuration()) * 100;
	}
</script>

<div class="flex flex-col items-center max-w-prose w-full">
	<!-- svelte-ignore a11y-click-events-have-key-events -->
	<!-- svelte-ignore a11y-no-static-element-interactions -->
	<div id="waveform" class="w-full h-fit"></div>

	<button
		on:click={togglePlay}
		class="btn btn-ghost aspect-square p-1 mt-4 fa-solid text-4xl justify-center {isPlaying
			? 'fa-pause'
			: 'fa-play'}"
	>
	</button>
</div>
