<script>
	import FaRegQuestionCircle from 'svelte-icons/fa/FaRegQuestionCircle.svelte';
	import FaAlignLeft from 'svelte-icons/fa/FaAlignLeft.svelte';
	import GiPianoKeys from 'svelte-icons/gi/GiPianoKeys.svelte';

	import { onMount } from 'svelte';

	let file = null;
	let fileId = '';
	let userId = '';

	let youtubeUrl = '';

	let uploading = false;
	let uploaded = false;
	let warning = '';
	let urlWarn = '';

	const ENDPOINT = 'http://127.0.0.1:5000';

	onMount(() => {
		document.addEventListener('dragover', handleDragOver);
		document.addEventListener('drop', handleDrop);

		if (!userId) {
			// Generate a random user ID and store it in a cookie
			document.cookie = `userId=${Math.random().toString(36).substring(2, 15)}`;

			const cookies = document.cookie.split(';');
			for (const cookie of cookies) {
				const [name, value] = cookie.trim().split('=');
				if (name === 'userId') {
					userId = value;
				}
			}
		}

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

	const MAX_FILE_SIZE_MB = 100;

	const handleURL = async () => {
		urlWarn = '';

		const regex =
			/^(?:https?:\/\/)?(?:www\.)?(?:youtube\.com\/(?:[^\/\n\s]+\/\S+\/|(?:v|e(?:mbed)?)\/|\S*?[?&]v=)|youtu\.be\/)([a-zA-Z0-9_-]{11})/;

		const match = youtubeUrl.match(regex);
		const videoId = match ? match[1] : null;

		try {
			uploading = true;
			const response = await fetch(`${ENDPOINT}/process_video`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					videoId: videoId,
					userId: userId
				})
			});

			if (response.ok) {
				const result = await response.json();
				uploaded = true;
				fileId = result.file_id;
				console.log('File uploaded successfully!: ', result.file_id);
			} else {
				console.error('Error:', response.statusText);
			}
		} catch (error) {
			console.error('Error:', error.message);
		} finally {
			uploading = false;
		}
	};

	const uploadFile = async () => {
		if (file) {
			const formData = new FormData();
			formData.append('file', file);

			try {
				uploading = true; // Set uploading to true when starting the upload
				const response = await fetch(`${ENDPOINT}/upload?user_id=${userId}`, {
					method: 'POST',
					body: formData
				});

				if (response.ok) {
					const result = await response.json();
					uploaded = true;
					fileId = result.file_id;
					console.log('File uploaded successfully!: ', result.file_id);
				} else {
					console.error('Error uploading file:', response.statusText);
				}
			} catch (error) {
				console.error('Error uploading file:', error.message);
			} finally {
				uploading = false;
			}
		} else {
			warning = 'No file selected!';
			console.warn(warning);
		}
	};

	const getFile = async (func = 'send_file') => {
		if (fileId) {
			try {
				const response = await fetch(`${ENDPOINT}/${func}/${userId}?fileId=${fileId}`);

				if (response.ok) {
					const blob = await response.blob();
					const url = window.URL.createObjectURL(blob);

					const link = document.createElement('a');
					link.href = url;
					link.download = `${fileId}`;
					link.click();

					window.URL.revokeObjectURL(url);
				} else {
					console.error('Error:', response.statusText);
				}
			} catch (error) {
				console.error('Error:', error.message);
			}
		} else {
			warning = 'No file uploaded!';
			console.warn(warning);
		}
	};
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
{#if uploading}
	<p class="text-3xl m-4">Uploading...</p>
{:else if !uploaded}
	<div class="flex">
		<input
			type="text"
			bind:value={youtubeUrl}
			placeholder="Enter a YouTube URL"
			class="input input-bordered w-full max-w-sm mx-2"
		/>
		{#if urlWarn != ''}
			<button class="btn mx-2 tooltip tooltip-open" data-tip={urlWarn} on:click={handleURL}
				>Upload</button
			>
		{:else}
			<button class="btn mx-2" on:click={handleURL}>Upload</button>
		{/if}
	</div>
	<h2 class="my-4">OR</h2>
	<div
		class="border border-dashed bg-base-200 border-neutral relative rounded-lg dropzone"
		on:click={handleClick}
		on:drop={handleDrop}
	>
		<input class="cursor-pointer relative block opacity-0 w-full max-w-lg h-full p-20 z-50" />
		<div
			class="text-center absolute top-1/3 bottom-1/3 left-0 right-0 flex items-center justify-center"
		>
			{#if warning != ''}
				<p class="text-center">{warning}</p>
			{:else if file}
				<p class="text-center">Selected file: {file.name}</p>
			{:else}
				<h4>Click or drop a file here</h4>
			{/if}
		</div>
	</div>
	<button class="btn m-4" on:click={uploadFile}>Upload</button>
{:else}
	<p class="text-3xl m-4">File Uploaded!</p>
	<div class="flex my-4">
		<button class="btn m-1 p-2 tooltip" data-tip="Track information" on:click={getFile}>
			<FaRegQuestionCircle />
		</button>
		<button class="btn m-1 p-2 tooltip" data-tip="Split instruments" on:click={getFile}>
			<FaAlignLeft />
		</button>
		<button class="btn m-1 p-2 tooltip" data-tip="Convert to MIDI" on:click={getFile}>
			<GiPianoKeys />
		</button>
	</div>
{/if}
