<script lang="ts">
	import '../app.postcss'
	import { page } from '$app/stores'
	import { pwaInfo } from 'virtual:pwa-info'
	import { onMount } from 'svelte'
	import InstallPrompt from '$components/InstallPrompt.svelte'

	$: webManifestLink = pwaInfo ? pwaInfo.webManifest.linkTag : ''

	let ReloadPrompt: typeof import('$lib/components/ReloadPrompt.svelte')['default'] | null = null
	let deferredPrompt: any
	let pwaInstallable = false
	let pwaInstalled = false

	onMount(async () => {
		pwaInfo && (ReloadPrompt = (await import('$lib/components/ReloadPrompt.svelte')).default)

		window.addEventListener('DOMContentLoaded', async (event) => {
			if ('BeforeInstallPromptEvent' in window) {
				console.log('⏳ BeforeInstallPromptEvent supported but not fired yet')
			} else {
				console.log('❌ BeforeInstallPromptEvent NOT supported')
			}
		})

		window.addEventListener('beforeinstallprompt', (e) => {
			// Prevents the default mini-infobar or install dialog from appearing on mobile
			e.preventDefault()
			// Save the event because you’ll need to trigger it later.
			deferredPrompt = e
			pwaInstallable = true
			// Show your customized install prompt for your PWA
			console.log('✅ BeforeInstallPromptEvent fired', true)
		})

		window.addEventListener('appinstalled', (e) => {
			pwaInstalled = true
			console.log('✅ AppInstalled fired', true)
		})
	})
</script>

<svelte:head>
	<title>亿点</title>
	{@html webManifestLink}
</svelte:head>

<div class="h-[100vh] flex flex-col items-center max-w-4xl mx-auto">
	<div class="text-2xl font-bold px-2 py-4 flex items-center justify-between w-full">
		<a href="about" class="text-sm p-4">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				width="16"
				height="16"
				fill="currentColor"
				class="bi bi-info-circle"
				viewBox="0 0 16 16"
			>
				<path d="M8 15A7 7 0 1 1 8 1a7 7 0 0 1 0 14zm0 1A8 8 0 1 0 8 0a8 8 0 0 0 0 16z" />
				<path
					d="m8.93 6.588-2.29.287-.082.38.45.083c.294.07.352.176.288.469l-.738 3.468c-.194.897.105 1.319.808 1.319.545 0 1.178-.252 1.465-.598l.088-.416c-.2.176-.492.246-.686.246-.275 0-.375-.193-.304-.533L8.93 6.588zM9 4.5a1 1 0 1 1-2 0 1 1 0 0 1 2 0z"
				/>
			</svg>
		</a>
		<h1>亿点问</h1>
		{#if pwaInstallable && !pwaInstalled}
			<InstallPrompt {deferredPrompt} />
		{:else}
			<span />
		{/if}
	</div>
	<slot />
	{#if $page.route.id === '/about'}
		<div class="h-full w-full grid place-items-center">
			<button class="p-4" on:click={() => history.back()}>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="w-6 h-6"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M10.5 19.5L3 12m0 0l7.5-7.5M3 12h18"
					/>
				</svg>
			</button>
		</div>
	{/if}
</div>

{#if ReloadPrompt}
	<svelte:component this={ReloadPrompt} />
{/if}
