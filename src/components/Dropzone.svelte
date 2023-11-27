<script>
	import { onMount } from 'svelte';

	export let file = null;
	export let warning = '';

	const MAX_FILE_SIZE_MB = 100;

	onMount(() => {
		document.addEventListener('dragover', handleDragOver);
		document.addEventListener('drop', handleDrop);

		return () => {
			document.removeEventListener('dragover', handleDragOver);
			document.removeEventListener('drop', handleDrop);
		};
	});

	const handleDragOver = (event) => {
		event.preventDefault();
	};

	const handleDrop = (event) => {
		event.preventDefault();
		const droppedFile = event.dataTransfer.files[0];
		if (droppedFile) {
			checkFile(droppedFile);
		}
	};

	const handleClick = () => {
		const input = document.createElement('input');
		input.type = 'file';
		input.multiple = true;
		input.accept = 'audio/';
		input.addEventListener('change', handleFileSelect);
		input.click();
	};

	const handleFileSelect = (event) => {
		const selectedFile = event.target.files[0];
		if (selectedFile) {
			checkFile(selectedFile);
		}
	};

	const checkFile = (checkedFile) => {
		if (checkedFile.size > MAX_FILE_SIZE_MB * 1024 * 1024) {
			warning = 'File size exceeds the maximum allowed (100MB)!';
			console.warn(warning);
			return;
		}

		if (!checkedFile.type.startsWith('audio/')) {
			warning = 'Selected file is not an audio file!';
			console.warn(warning);
			return;
		}

		warning = '';

		setFile(checkedFile);
	};

	const setFile = (newFile) => {
		file = newFile;
	};
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
	class="border border-dashed bg-base-200 border-neutral relative rounded-lg dropzone"
	on:click={handleClick}
	on:drop={handleDrop}
>
	<input class="cursor-pointer relative block opacity-0 w-full max-w-lg h-full p-20 z-10" />
	<div
		class="text-center absolute top-1/3 bottom-1/3 left-0 right-0 flex items-center justify-center"
	>
		{#if file}
			<p class="text-center p-2">Selected file: {file.name}</p>
		{:else}
			<h4>Click or drop a file here</h4>
		{/if}
	</div>
</div>
