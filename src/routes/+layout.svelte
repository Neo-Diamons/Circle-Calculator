<script lang="ts">
	import '../app.css';
	import { onMount } from 'svelte';
	import favicon from '$lib/assets/favicon.ico';
	import { m } from '$lib/paraglide/messages.js';
	import { getLocale, setLocale } from '$lib/paraglide/runtime';

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

<header class="p-4 border-b border-gray-300 flex items-center justify-between">
	<div>
		<h1 class="text-2xl font-bold">{m.title()}</h1>
	</div>

	<div class="flex items-center ml-4">
		<select
			value={getLocale()}
			onchange={(e) => setLocale((e.target as HTMLSelectElement).value as "en" | "fr")}
			class="rounded border border-gray-300 p-2"
		>
			<option value="en">🇬🇧 English</option>
			<option value="fr">🇫🇷 Français</option>
		</select>
	</div>
</header>

{@render children?.()}
