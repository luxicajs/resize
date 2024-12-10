<script lang="ts">
	import Cropper from 'cropperjs';
	import 'cropperjs/dist/cropper.css';
	import { onDestroy, onMount } from 'svelte';

	import * as Menubar from '$lib/components/ui/menubar/index.js';
	import * as Dialog from '$lib/components/ui/dialog/index.js';
	import { Slider } from '$lib/components/ui/slider/index.js';
	import { Input } from '$lib/components/ui/input/index.js';
	import * as Select from '$lib/components/ui/select/index.js';
	import { Button } from '$lib/components/ui/button/index.js';
	import Compressor from 'compressorjs';

	let element: HTMLImageElement;
	let cropper: Cropper;
	let filePicker: HTMLInputElement;

	let exportDialogOpen = $state(false);

	onMount(() => {
		cropper = new Cropper(element, {
			viewMode: 1
		});
	});

	onDestroy(() => {
		if (cropper) {
			cropper.destroy();
		}
	});

	function openFile(event: any) {
		const file = event.target.files[0];

		const reader = new FileReader();
		reader.onload = (e: any) => {
			cropper.replace(e.target?.result);
		};
		reader.readAsDataURL(file);
	}

	let exportW = $state(0); // Export width
	let exportH = $state(0); // Export height
	let exportQ = $state(100); // Export quality
	let exportF = $state('image/png'); // Export format

	let exportImg: HTMLImageElement;
	let blob: Blob;
	let cropperData: any;
	async function initExport() {
		const c = cropper.getCroppedCanvas();
		cropperData = c;
		c.toBlob((res) => {
			if (!res) return alert('error');
			exportW = c.width;
			exportH = c.height;
			exportQ = 100;
			blob = res;
			exportDialogOpen = true;
		}, 'image/png');
	}

	$effect(() => {
		if (exportDialogOpen === true && blob) {
			new Compressor(blob, {
				width: exportW,
				height: exportH,
				quality: exportQ / 100,
				mimeType: exportF,
				success(result) {
					exportImg.src = window.URL.createObjectURL(result);
				}
			});
		}
	});
</script>

<!-- Export Dialog -->

<Dialog.Root open={exportDialogOpen} onOpenChange={() => (exportDialogOpen = !exportDialogOpen)}>
	<Dialog.Content class="max-w-[75%]">
		<Dialog.Header>
			<Dialog.Title>Export</Dialog.Title>
		</Dialog.Header>
		<div class="flex flex-col 2xl:flex-row">
			<img bind:this={exportImg} src="" class="rounded-md max-w-[640px] max-h-[360px]" alt="Exported img" />
			<div class="flex grow flex-col justify-center">
				<div class="flex flex-row items-center gap-4 p-4">
					Width:
					<Input type="number" bind:value={exportW} />
				</div>

				<div class="flex flex-row items-center gap-4 p-4">
					Height:
					<Input type="number" bind:value={exportH} />
				</div>

				<div class="flex flex-row items-center gap-4 p-4">
					Quality:
					<Slider value={[exportQ]} max={100} onValueChange={(v) => (exportQ = v[0])} />
					<Input type="number" bind:value={exportQ} />
				</div>

				<div class="flex flex-row items-center gap-4 p-4">
					Format:
					<Select.Root type="single" bind:value={exportF}>
						<Select.Trigger>{exportF}</Select.Trigger>
						<Select.Content>
							<Select.Item value="image/png">image/png</Select.Item>
							<Select.Item value="image/jpeg">image/jpeg</Select.Item>
							<Select.Item value="image/webp">image/webp</Select.Item>
						</Select.Content>
					</Select.Root>
				</div>
			</div>
		</div>
		<Dialog.Footer>
			<Button onclick={() => window.open(exportImg.src)}>Export</Button>
		</Dialog.Footer>
	</Dialog.Content>
</Dialog.Root>

<input type="file" class="hidden" bind:this={filePicker} onchange={(ev) => openFile(ev)} />
<div class="flex h-screen flex-col">
	<Menubar.Root>
		<Menubar.Menu>
			<Menubar.Trigger>File</Menubar.Trigger>
			<Menubar.Content>
				<Menubar.Item onclick={() => filePicker.click()}>Open</Menubar.Item>
				<Menubar.Item onclick={() => initExport()}>Export</Menubar.Item>
			</Menubar.Content>
		</Menubar.Menu>
	</Menubar.Root>
	<div class="w-screen grow h-[90%]">
		<img bind:this={element} alt="Editor" src="/sample.webp" />
	</div>
</div>

<style>
	img {
		display: block;
		max-width: 100%;
	}
</style>
