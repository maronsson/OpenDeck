<script lang="ts">
	import type { DeviceInfo } from "$lib/DeviceInfo";
	import type { Profile } from "$lib/Profile";

	import { initPortBase } from "$lib/ports";
	import { inspectedParentAction } from "$lib/propertyInspector";
	import { actionList, deviceSelector, halfSize, profileManager } from "$lib/singletons";

	import ActionList from "../components/ActionList.svelte";
	import DeviceSelector from "../components/DeviceSelector.svelte";
	import DeviceView from "../components/DeviceView.svelte";
	import NoDevicesDetected from "../components/NoDevicesDetected.svelte";
	import ParentActionView from "../components/ParentActionView.svelte";
	import PluginManager from "../components/PluginManager.svelte";
	import ProfileManager from "../components/ProfileManager.svelte";
	import PropertyInspectorView from "../components/PropertyInspectorView.svelte";
	import SettingsView from "../components/SettingsView.svelte";

	let devices: { [id: string]: DeviceInfo } = {};
	let selectedDevice: string;
	let selectedProfiles: { [id: string]: Profile } = {};

	initPortBase();
</script>

<svelte:window on:dragover={(event) => event.preventDefault()} on:drop={(event) => event.preventDefault()} />

<div class="flex flex-row h-screen max-h-screen overflow-hidden">
	<div class="flex flex-col grow min-w-0 min-h-0">
		<nav class="flex flex-row flex-wrap gap-y-2 justify-between items-center p-3" class:hidden={$inspectedParentAction}>
			<div class="flex flex-col items-start space-y-1">
				<DeviceSelector bind:devices bind:value={selectedDevice} bind:selectedProfiles bind:this={$deviceSelector} />
				{#key selectedDevice}
					{#if selectedDevice && devices[selectedDevice]}
						<ProfileManager bind:device={devices[selectedDevice]} bind:profile={selectedProfiles[selectedDevice]} bind:this={$profileManager} />
					{/if}
				{/key}
			</div>

			<div class="flex flex-row items-center space-x-2" class:mr-4={Object.keys(devices).length > 0}>
				<button
					class="px-3 py-1 text-sm text-neutral-300 border border-neutral-600 rounded-lg transition-colors"
					class:bg-neutral-700={!$halfSize}
					class:bg-blue-600={$halfSize}
					on:click={() => halfSize.update((v) => !v)}
				>
					50%
				</button>
				<PluginManager />
				<SettingsView />
			</div>
		</nav>

		{#if Object.keys(devices).length > 0 && selectedProfiles}
			{#if $inspectedParentAction}
				<ParentActionView bind:profile={selectedProfiles[selectedDevice]} />
			{/if}

			{#each Object.entries(devices) as [id, device]}
				{#if device && selectedProfiles[id]}
					<DeviceView bind:device bind:profile={selectedProfiles[id]} bind:selectedDevice />
				{/if}
			{/each}

			{#if selectedProfiles[selectedDevice]}
				<PropertyInspectorView bind:device={devices[selectedDevice]} bind:profile={selectedProfiles[selectedDevice]} />
			{/if}
		{:else}
			<NoDevicesDetected />
		{/if}
	</div>

	<ActionList bind:this={$actionList} />
</div>
