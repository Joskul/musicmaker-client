<script>
	import { onMount } from 'svelte';
	import Dropzone from './Dropzone.svelte';
	import AudioPlayer from './AudioPlayer.svelte';

	let file = Dropzone.file;
	let dropzoneWarning = Dropzone.warning;

	let youtubeUrl = '';
	let uploading = false;
	let uploaded = false;
	let urlWarn = '';

	let processId = '';
	let userId = '';

	const ENDPOINT = 'http://127.0.0.1:5000';

	onMount(() => {
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
	});

	const handleURL = async () => {
		urlWarn = '';
		if (youtubeUrl === '') {
			urlWarn = 'Please enter a URL';
			return;
		}

		const regex =
			/^(?:https?:\/\/)?(?:www\.)?(?:youtube\.com\/(?:[^\/\n\s]+\/\S+\/|(?:v|e(?:mbed)?)\/|\S*?[?&]v=)|youtu\.be\/)([a-zA-Z0-9_-]{11})/;

		const match = youtubeUrl.match(regex);
		const videoId = match ? match[1] : null;

		try {
			uploading = true;
			const response = await fetch(`${ENDPOINT}/process_url`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					videoId: videoId,
					userId: userId,
					format: 'youtube'
				})
			});

			if (response.ok) {
				const result = await response.json();
				uploaded = true;
				processId = result.process_id;
				console.log('File uploaded successfully! Process ID: ', result.process_id);
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
					processId = result.process_id;
					console.log('File uploaded successfully! Process ID : ', result.process_id);
				} else {
					console.error('Error uploading file:', response.statusText);
				}
			} catch (error) {
				console.error('Error uploading file:', error.message);
			} finally {
				uploading = false;
			}
		} else {
			dropzoneWarning = 'No file selected!';
			console.warn(dropzoneWarning);
		}
	};

	const processFile = async (func = 'get_file') => {
		if (processId) {
			try {
				const response = await fetch(`${ENDPOINT}/${func}/${userId}?file_id=${processId}`);

				if (response.ok) {
					const blob = await response.blob();
					const url = window.URL.createObjectURL(blob);

					const link = document.createElement('a');
					link.href = url;
					link.download = `${processId}`;
					link.click();

					window.URL.revokeObjectURL(url);
				} else {
					console.error('Error:', response.statusText);
				}
			} catch (error) {
				console.error('Error:', error.message);
			}
		} else {
			dropzoneWarning = 'No file uploaded!';
			console.warn(dropzoneWarning);
		}
	};

	let actions = [
		{
			label: 'Track Information',
			action: 'send_file',
			icon: 'fa-circle-info'
		},
		{
			label: 'Split Instruments',
			action: 'send_file',
			icon: 'fa-guitar'
		},
		{
			label: 'Convert to MIDI',
			action: 'send_file',
			icon: 'fa-piano-keyboard'
		}
	];
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

	<Dropzone bind:file bind:warning={dropzoneWarning} />
	{#if dropzoneWarning != ''}
		<button class="btn m-4 tooltip tooltip-open" data-tip={dropzoneWarning} on:click={uploadFile}
			>Upload</button
		>
	{:else}
		<button class="btn m-4" on:click={uploadFile}>Upload</button>
	{/if}
{:else}
	<AudioPlayer src="{ENDPOINT}/get_file/{userId}" />
	<div class="flex my-4 justify-center">
		{#each actions as item}
			<button
				class="btn m-1 p-5 tooltip aspect-square h-full"
				data-tip={item.label}
				on:click={() => processFile(item.action)}
			>
				<i class="fa-solid {item.icon} text-6xl"></i>
			</button>
		{/each}
	</div>
{/if}
