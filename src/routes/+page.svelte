<script lang="ts">
	import Canvas from '$lib/components/canvas.svelte';
	import { browser } from '$app/environment';
	import { enhance } from '$app/forms';
	import type { PinResponse } from '$lib/types';
	import { scrollPosition, all_pins } from '$lib/stores';
	// $: console.log($all_pins);
	export let data;
	let pins: PinResponse[] | undefined = data.pins;
	let new_pins: PinResponse[] | undefined = pins;
	let images: HTMLImageElement[] = [];
	let gap: number = 15;
	$: {
		if (new_pins && new_pins.length > 0 && browser) {
			new_pins.forEach((pin, i) => {
				const img = new Image();
				img.src = pin.images['orig'].url;
				img.id = i.toString();
				img.width = pin.images['orig'].width;
				img.height = pin.images['orig'].height;
				images.push(img);
			});
		}
	}
	$: bookmark = data.bookmark;
	let btn: HTMLButtonElement;
	let y = 0;
	let touchStart: number = 0;
	$: $scrollPosition = y;

	function refetch() {
		btn.click();
	}

	function onKeyDown(e: KeyboardEvent) {
		if (!pins) return;
		if (e.key == 'ArrowUp' && y == 0) return;
		if (e.key == 'ArrowDown' && y == pins.length - 1) return;
		if (e.key == 'ArrowDown' && y == pins.length - gap) {
			y = pins.length - gap + 1;
			// console.log('pinslength:', pins.length, 'y:', y, 'refetching');
			refetch();
		} else if (e.key == 'ArrowDown') y += 1;
		if (e.key == 'ArrowUp') y -= 1;
	}
</script>

<svelte:window on:keydown={onKeyDown} />
{#if pins}
	{#if pins.length > 0}
		<div
			class="w-screen h-screen overflow-hidden"
			on:wheel={(e) => {
				if (!pins) return;
				if (e.deltaY <= 0 && y == 0) return;
				if (e.deltaY >= 0 && y >= pins.length - 1) return;
				if (e.deltaY >= 0 && y == pins.length - gap) {
					y = pins.length - gap + 1;
					// console.log('pinslength:', pins.length, 'y:', y, 'refetching');
					refetch();
				} else if (e.deltaY >= 0 && y !== pins.length - gap) y += 1;
				if (e.deltaY <= 0 && y > 0) y -= 1;
			}}
			on:touchstart={(e) => {
				touchStart = e.changedTouches[0].clientY;
			}}
			on:touchmove={(e) => {
				if (!pins) return;
				const touch = e.changedTouches[0];
				const direction = touchStart < touch.clientY ? 'up' : 'down';
				if (direction == 'up' && y == 0) return;
				if (direction == 'down' && y >= pins.length - 1) return;
				if (direction == 'down' && y == pins.length - gap) {
					y = pins.length - gap + 1;
					// console.log('pinslength:', pins.length, 'y:', y, 'refetching');
					refetch();
				} else if (direction == 'down' && y !== pins.length - gap) y += 1;
				if (direction == 'up' && y > 0) y -= 1;
			}}
		>
			<form
				method="post"
				use:enhance={() => {
					return async ({ result }) => {
						if (!pins) return;
						// @ts-ignore
						if (!result.data) return;
						// @ts-ignore
						new_pins = result.data.new_pins;
						// @ts-ignore
						pins = [...pins, ...result.data.new_pins];
						// @ts-ignore
						bookmark = result.data.new_bookmark;
						// console.log('refetched');
					};
				}}
				action="?/refetch"
			>
				<input type="text" class="hidden" name="bookmark" value={bookmark} />
				<button bind:this={btn} type="submit" class="hidden" />
			</form>
			{#if pins[y] && images.length > 0}
				<Canvas {images} />
			{/if}
		</div>
	{:else}
		<div class="flex justify-center items-center h-screen">
			<p class="text-white">no data found</p>
		</div>
	{/if}
{/if}
