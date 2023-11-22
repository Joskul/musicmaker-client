<script>
	import FaGithub from 'svelte-icons/fa/FaGithub.svelte';
	import FaTools from 'svelte-icons/fa/FaTools.svelte';
	import FaHome from 'svelte-icons/fa/FaHome.svelte';

	let drawerItems = [
		{
			title: 'Home',
			child: FaHome,
			url: '/'
		},
		{
			title: 'Splitter',
			child: FaTools,
			url: '/tool'
		},
		{
			title: 'GitHub',
			child: FaGithub,
			url: 'https://github.com/Joskul/musicmaker-client'
		}
	];
</script>

<ul class="menu absolute bottom-0 left-0 m-8 bg-base-300 rounded-lg">
	{#each drawerItems as item}
		<li>
			<a
				class="tooltip tooltip-right h-16 my-2 py-3.5 bg-base-100 aspect-square"
				data-tip={item.title}
				href={item.url}
			>
				<svelte:component this={item.child} />
			</a>
		</li>
	{/each}
</ul>
