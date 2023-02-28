<script>
    import '@skeletonlabs/skeleton/themes/theme-skeleton.css';
    import '@skeletonlabs/skeleton/styles/all.css';
	import "@fontsource/open-sans";
	import '../app.postcss';
	import { AppShell, AppBar, LightSwitch } from '@skeletonlabs/skeleton'
</script>

<AppShell regionPage="dark:bg-surface-900 print:bg-white">
	<svelte:fragment slot="header">
		<AppBar gridColumns="grid-cols-1" slotDefault="place-self-center" background="bg-surface-300 dark:bg-surface-800 print:bg-white">
			<a class="text-xl font-medium" href="/">Επιλογή Τρίμηνης Κλινικής</a>
		</AppBar>
	</svelte:fragment>
	<slot />
	<svelte:fragment slot="footer">
		<AppBar class="print:hidden" background="bg-surface-300 dark:bg-surface-800">
			<svelte:fragment slot="lead"><p>Από τον <a class="text-secondary-800 dark:text-secondary-50 font-bold hover:underline" href="https://chrisch.dev">chrisch</a></p></svelte:fragment>
			<svelte:fragment slot="trail"><LightSwitch height="h-6"/></svelte:fragment>
		</AppBar>
	</svelte:fragment>
</AppShell>