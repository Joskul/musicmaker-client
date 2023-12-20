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

	let processName = '';
	let processId = '';
	let userId = '';

	const ENDPOINT = 'https://beat-ml-api.onrender.com';

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
			const response = await fetch(`${ENDPOINT}/process-url?user_id=${userId}`, {
				method: 'POST',
				headers: {
					'Content-Type': 'application/json'
				},
				body: JSON.stringify({
					video_id: videoId,
					format: 'youtube'
				})
			});

			if (response.ok) {
				const result = await response.json();
				uploaded = true;
				processId = result.process_id;
				processName = result.process_name;
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
				const response = await fetch(`${ENDPOINT}/process-file?user_id=${userId}`, {
					method: 'POST',
					body: formData
				});

				if (response.ok) {
					const result = await response.json();
					uploaded = true;
					processId = result.process_id;
					processName = result.process_name;
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

	const processAction = async (
		api_endpoint = 'audio-file',
		file_format = 'mp3',
		download = true
	) => {
		console.log(processId);
		if (processId != '') {
			try {
				const response = await fetch(
					`${ENDPOINT}/${api_endpoint}/${userId}?process_id=${processId}`,
					{
						method: 'GET',
						headers: {
							'Content-Type': 'application/json'
						}
					}
				);

				if (response.ok) {
					if (download) {
						const blob = await response.blob();
						const url = window.URL.createObjectURL(blob);

						const link = document.createElement('a');
						link.href = url;
						link.download = `${processName}.${file_format}`;
						link.click();

						window.URL.revokeObjectURL(url);
					}
					const result = await response.json();
					return result;
				} else {
					console.error('Error:', response.statusText);
					throw response.statusText;
				}
			} catch (error) {
				console.error('Error:', error.message);
				throw error.message;
			}
		} else {
			dropzoneWarning = 'No file uploaded!';
			console.warn(dropzoneWarning);
			throw dropzoneWarning;
		}
	};

	let actions = [
		{
			label: 'Split Instruments',
			action: ['audio-file', 'mp3'],
			icon: 'fa-guitar'
		},
		{
			label: 'Convert to MIDI',
			action: ['audio-file', 'mid'],
			icon: 'fa-piano-keyboard'
		}
	];
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
{#if uploading}
	<h3 class="m-4">
		Uploading
		<div class="loading loading-dots" />
	</h3>
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

	<h4 class="my-4">OR</h4>

	<Dropzone bind:file bind:warning={dropzoneWarning} />
	{#if dropzoneWarning != ''}
		<button class="btn m-4 tooltip tooltip-open" data-tip={dropzoneWarning} on:click={uploadFile}
			>Upload</button
		>
	{:else}
		<button class="btn m-4" on:click={uploadFile}>Upload</button>
	{/if}
{:else}
	<div>
		<h3>{processName}</h3>
	</div>
	<AudioPlayer src="{ENDPOINT}/audio-file/{userId}" />
	<div>
		<h3 class="my-4">
			Track Identification:
			{#await processAction('info', '', false)}
				<div class="loading loading-dots" />
			{:then type}
				{type.text}
			{:catch error}
				{error}
			{/await}
		</h3>
	</div>
	<div class="flex my-4 justify-center">
		{#each actions as item}
			<button
				class="btn m-1 p-5 tooltip aspect-square h-full"
				data-tip={item.label}
				on:click={() => processAction(...item.action)}
			>
				<i class="fa-solid {item.icon} md:text-6xl text-2xl"></i>
			</button>
		{/each}
	</div>
{/if}
