<script lang="ts">
  import "./app.css"
	import { bmpPalette } from "./lib/bmp"

	let files: FileList | null | undefined = $state(undefined)
	let canvas: HTMLCanvasElement

	let bufferPromise = $derived.by(() => {
		if (!files || files?.length < 1) return undefined

		return files[0].arrayBuffer()
	})

	let buffer: ArrayBuffer | undefined = $state(undefined)

	$effect(() => {
		if (bufferPromise) {
			Promise.resolve(bufferPromise).then((b) => (buffer = b))
		}
	})

	let palette = $derived.by(() => {
		if (!buffer) return undefined
		return bmpPalette(buffer)
	})

	let base64 = $derived.by(() => {
		if (!palette) return undefined

		const ctx = canvas.getContext("2d")
		if (!ctx) return undefined

		ctx.clearRect(0, 0, canvas.width, canvas.height)
		for (let i = 0; i < palette.length; i++) {
			ctx.fillStyle = `rgb(${palette[i].red}, ${palette[i].green}, ${palette[i].blue})`
			ctx.fillRect(i, 0, 1, 1)
		}

		return canvas.toDataURL("image/png")
	})
</script>

<div style="width: 100%; text-align: center;">
<h2>LnzLive bitmap palette converter</h2>
<p>This utility converts petz 4 compatible palettes to a png that is compatible with <a href="https://github.com/Draconizations/LnzLive/tree/add-palette-swaps">my fork of lnzlive</a>.<br/>
Upload the image here and download the image that it displays.
</p>

	<b>Upload bitmap palette</b><br />
	<input type="file" bind:files accept=".bmp" /><br />
	<canvas style="display: none;" bind:this={canvas} height={1} width={256}></canvas>
	{#if base64}
		<br />
		<img src={base64} alt="your petz palette" style="height: 24px; width: 100%; image-rendering: pixelated;" /><br/>
		<small>Don't worry about the image being stretched! Downloading will result in the correct proportions.</small>
	{/if}
</div>
