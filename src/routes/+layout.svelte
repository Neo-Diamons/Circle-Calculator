<script lang="ts">
	import '../app.css';
	import { onMount } from 'svelte';
	import favicon from '$lib/assets/favicon.ico';
	import { ModeWatcher, toggleMode, mode } from 'mode-watcher';
	import { m } from '$lib/paraglide/messages.js';
	import { getLocale, setLocale } from '$lib/paraglide/runtime';

	import Sun from '@lucide/svelte/icons/sun';
	import Moon from '@lucide/svelte/icons/moon';

	let { children } = $props();

	async function detectSWUpdate() {
		const registration = await navigator.serviceWorker.ready;

		registration.addEventListener('updatefound', () => {
			const newSW = registration.installing;
			newSW?.addEventListener('statechange', () => {
				if (newSW.state === 'installed') {
					if (confirm(m.new_version_available())) {
						newSW?.postMessage({ type: 'SKIP_WAITING' });
						window.location.reload();
					}
				}
			});
		});
	}

	onMount(() => {
		detectSWUpdate();
	});
</script>

<svelte:head>
	<link rel="icon" href={favicon} />
	<title>{m.title()}</title>
	<link rel="manifest" href="/manifest-{getLocale()}.json" />
</svelte:head>

<ModeWatcher />

<header class="flex items-center justify-between border-b bg-black/5 p-4">
	<div>
		<h1 class="text-2xl font-bold">{m.title()}</h1>
	</div>

	<div class="ml-4 flex items-center">
		<select
			value={getLocale()}
			onchange={(e) => setLocale((e.target as HTMLSelectElement).value as 'en' | 'fr')}
			class="rounded border p-2"
		>
			<option value="en">🇬🇧 English</option>
			<option value="fr">🇫🇷 Français</option>
		</select>

		<button onclick={toggleMode} class="ml-4 rounded border p-2">
			{#if mode.current === 'light'}
				<Sun class="size-6" />
			{:else}
				<Moon class="size-6" />
			{/if}
		</button>
	</div>
</header>

{@render children?.()}
