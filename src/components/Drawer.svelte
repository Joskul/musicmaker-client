<script>
	import FaGithub from 'svelte-icons/fa/FaGithub.svelte';
	import FaTools from 'svelte-icons/fa/FaTools.svelte';
	import FaHome from 'svelte-icons/fa/FaHome.svelte';

	let drawerItems = [
		{
			title: 'Home',
			child: FaHome,
			icon: 'fa-solid fa-house',
			url: '/'
		},
		{
			title: 'Splitter',
			child: FaTools,
			icon: 'fa-solid fa-toolbox',
			url: '/tool'
		},
		{
			title: 'GitHub',
			child: FaGithub,
			icon: 'fa-brands fa-github',
			url: 'https://github.com/Joskul/musicmaker-client'
		}
	];
</script>

<ul class="menu absolute bottom-0 left-0 m-8 px-2 bg-base-300 rounded-lg z-50">
	{#each drawerItems as item}
		<li>
			<a
				class="tooltip tooltip-right my-1 bg-base-100 aspect-square flex justify-center"
				data-tip={item.title}
				href={item.url}
			>
				<i class="{item.icon} text-xl"></i>
			</a>
		</li>
	{/each}
</ul>
