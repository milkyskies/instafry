<script lang="ts">
	import RangeSlider from '$lib/components/range-slider.svelte';
	import { onMount } from 'svelte';
	import instructions from '$lib/assets/instructions.gif';
	import blank from '$lib/assets/blank.png';

	type FryParams = {
		saturation: number;
		contrast: number;
		noiseIntensity: number;
		jpegness: number;
		pixelation: number;
	};

	let imageSrc: string;
	let friedImageSrc: string;
	let friedBackgroundSrc: string;

	const fryParams = {
		saturation: 90,
		contrast: 50,
		noiseIntensity: 70,
		jpegness: 80,
		pixelation: 1
	};

	const maxJpegQuality = 100;

	let showParams = false;

	function handlePaste(event: ClipboardEvent) {
		const items = event.clipboardData?.items;
		if (!items) return;

		for (const item of items) {
			if (item.type.indexOf('image') === -1) continue;

			const blob = item.getAsFile();

			if (!blob) continue;

			const reader = new FileReader();

			reader.onload = async () => {
				imageSrc = reader.result as string;
				friedImageSrc = await deepFry({ ...fryParams, dataUrl: imageSrc });
			};

			reader.readAsDataURL(blob);
		}
	}

	async function deepFry({
		dataUrl,
		saturation,
		contrast,
		noiseIntensity,
		jpegness,
		pixelation
	}: FryParams & {
		dataUrl: string;
	}): Promise<string> {
		const img = new Image();
		img.src = dataUrl;

		await new Promise((res) => (img.onload = res)); // Wait for the image to load

		const canvas = document.createElement('canvas');
		const ctx = canvas.getContext('2d')!;

		canvas.width = img.width;
		canvas.height = img.height;

		// Pixelation
		const smallCanvas = document.createElement('canvas');
		smallCanvas.width = canvas.width / pixelation;
		smallCanvas.height = canvas.height / pixelation;
		const smallCtx = smallCanvas.getContext('2d')!;
		smallCtx.drawImage(img, 0, 0, smallCanvas.width, smallCanvas.height);
		ctx.drawImage(smallCanvas, 0, 0, canvas.width, canvas.height);

		const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
		const data = imageData.data;

		for (let i = 0; i < data.length; i += 4) {
			// Adjust saturation & contrast
			data[i] =
				data[i] < 128 ? Math.max(data[i] - contrast, 0) : Math.min(data[i] + saturation, 255);
			data[i + 1] =
				data[i + 1] < 128
					? Math.max(data[i + 1] - contrast, 0)
					: Math.min(data[i + 1] + saturation, 255);
			data[i + 2] =
				data[i + 2] < 128
					? Math.max(data[i + 2] - contrast, 0)
					: Math.min(data[i + 2] + saturation, 255);

			// Add noise
			const noise = (Math.random() * 2 - 1) * noiseIntensity;
			data[i] = clamp({ value: data[i] + noise, min: 0, max: 255 });
			data[i + 1] = clamp({ value: data[i + 1] + noise, min: 0, max: 255 });
			data[i + 2] = clamp({ value: data[i + 2] + noise, min: 0, max: 255 });
		}

		ctx.putImageData(imageData, 0, 0);

		// JPEG Artifacts
		return canvas.toDataURL('image/jpeg', (maxJpegQuality - jpegness) / maxJpegQuality);
	}

	function clamp({ value, min, max }: { value: number; min: number; max: number }): number {
		return Math.min(Math.max(value, min), max);
	}

	onMount(async () => {
		friedBackgroundSrc = await deepFry({
			dataUrl: blank,
			saturation: fryParams.saturation,
			contrast: fryParams.contrast,
			noiseIntensity: fryParams.noiseIntensity,
			jpegness: 90,
			pixelation: 2
		});

		document.addEventListener('paste', handlePaste);
	});

	async function reFry(): Promise<void> {
		friedImageSrc = await deepFry({
			dataUrl: imageSrc,
			saturation: fryParams.saturation,
			contrast: fryParams.contrast,
			noiseIntensity: fryParams.noiseIntensity,
			jpegness: fryParams.jpegness,
			pixelation: fryParams.pixelation
		});
	}
</script>

<div
	class="h-screen w-full flex items-center justify-center"
	style="background: url({friedBackgroundSrc}); background-repeat: no-repeat; background-size: cover;"
>
	{#if friedImageSrc}
		<figure class="h-[calc(100vh-60px)]">
			<img src={friedImageSrc} alt="deep fried" class="h-full w-full" />
		</figure>
	{:else}
		<img src={instructions} alt="instructions" class="h-full object-scale-down" />
	{/if}
</div>
{#if showParams}
	<div class="w-1/2 gap-2 flex flex-col">
		<div class="">Saturation: {fryParams.saturation}</div>
		<RangeSlider bind:value={fryParams.saturation} min={0} max={100} step={1} on:change={reFry} />
		<div class="">Contrast: {fryParams.contrast}</div>
		<RangeSlider bind:value={fryParams.contrast} min={0} max={100} step={1} on:change={reFry} />
		<div class="">Noise Intensity: {fryParams.noiseIntensity}</div>
		<RangeSlider
			bind:value={fryParams.noiseIntensity}
			min={0}
			max={100}
			step={1}
			on:change={reFry}
		/>
		<div class="">JPEG Quality: {fryParams.jpegness}</div>
		<RangeSlider bind:value={fryParams.jpegness} min={0} max={100} step={1} on:change={reFry} />
		<div class="">Pixelation: {fryParams.pixelation}</div>
		<RangeSlider bind:value={fryParams.pixelation} min={1} max={10} step={1} on:change={reFry} />
	</div>
{/if}
