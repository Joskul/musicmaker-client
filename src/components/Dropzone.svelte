<script>
	import { onMount } from 'svelte';

	let file = null;
	let uploaded = false;
	let fileName = '';
	let warning = '';

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
		input.accept = 'audio/'; // Add the file types you want to accept
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

	const uploadFile = async () => {
		if (file) {
			const formData = new FormData();
			formData.append('file', file);

			try {
				const response = await fetch('http://127.0.0.1:5000/upload', {
					method: 'POST',
					body: formData
				});

				if (response.ok) {
					uploaded = true;
					const result = await response.json();
					fileName = result.file_name;
					console.log('File uploaded successfully!: ', result.file_name);
				} else {
					console.error('Error uploading file:', response.statusText);
				}
			} catch (error) {
				console.error('Error uploading file:', error.message);
			}
		} else {
			warning = 'No file selected!';
			console.warn(warning);
		}
	};

	const getFile = () => {
		if (!window.ActiveXObject) {
			var save = document.createElement('a');
			save.href = 'http://127.0.0.1:5000/documents/' + fileName;
			save.target = '_blank';
			save.download = fileName || 'unknown';

			var evt = new MouseEvent('click', {
				view: window,
				bubbles: true,
				cancelable: false
			});
			save.dispatchEvent(evt);

			(window.URL || window.webkitURL).revokeObjectURL(save.href);
		}

		// for IE < 11
		else if (!!window.ActiveXObject && document.execCommand) {
			var _window = window.open('http://127.0.0.1:5000/documents/' + fileName, '_blank');
			_window.document.close();
			_window.document.execCommand(
				'SaveAs',
				true,
				fileName || 'http://127.0.0.1:5000/documents/' + fileName
			);
			_window.close();
		}
	};
</script>

<!-- svelte-ignore a11y-click-events-have-key-events -->
<!-- svelte-ignore a11y-no-static-element-interactions -->
{#if !uploaded}
	<div
		class="m-8 border border-dashed bg-base-200 border-neutral relative {$$props.class} dropzone"
		on:click={handleClick}
		on:drop={handleDrop}
	>
		<input class="cursor-pointer relative block opacity-0 w-full h-full p-20 z-50" />
		<div
			class="text-center absolute top-1/3 bottom-1/3 left-0 right-0 flex items-center justify-center m-4"
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
	<button class="btn" on:click={uploadFile}>Upload</button>
{:else}
	<p class="text-3xl">File Uploaded!</p>
	<button class="btn m-4" on:click={getFile}>Download</button>
{/if}
