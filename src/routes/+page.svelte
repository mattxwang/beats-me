<script>
	import { onMount } from 'svelte';
	import { guess } from 'web-audio-beat-detector';

	const DEFAULT_BEAT_PATH = '/qwerhacks.mp3';

	/**
	 * @type {AudioContext}
	 */
	let audioCtx;

	let bpm = 0,
		offset = 0,
		// tempo = 0,
		period = 0;
	let initialized = false,
		loaded = false,
		loading = false;

	let audioElement;
	/**
	 * @type {NodeJS.Timer | undefined}
	 */
	let beatInterval;
	let beatPath = DEFAULT_BEAT_PATH;
	let showBeat = false;

	/**
	 * @type {FileList}
	 */
	let files;

	onMount(init);

	function init() {
		initialized = true;
	}

	/**
	 * @param {{ bpm: any; offset: any; }} obj
	 */
	function finishProcessingSong(obj) {
		bpm = obj.bpm;
		offset = obj.offset;
		// type error with TS :(
		// tempo = obj.tempo;
		period = 1 / (obj.bpm / 60);
		loaded = true;
		loading = false;
	}

	function setup() {
		loading = true;
		if (!audioCtx) {
			const AudioContext = window.AudioContext;
			audioCtx = new AudioContext();
		}
		fetch(DEFAULT_BEAT_PATH)
			.then((data) => data.arrayBuffer())
			.then((buffer) => audioCtx.decodeAudioData(buffer))
			.then((audioBuffer) => guess(audioBuffer))
			.then((obj) => {
				finishProcessingSong(obj)
			})
			.catch((err) => console.error(err));
	}

	function startBeatCounter() {
		beatInterval = setInterval(() => {
			showBeat = true;
			setTimeout(() => {
				showBeat = false;
			}, 200);
		}, period * 1000);
	}

	function play() {
		setTimeout(startBeatCounter, offset * 1000);
	}

	function pause() {
		clearInterval(beatInterval);
	}

	/**
	 * @param {Blob} file
	 */
	function readAsArrayBuffer(file){
		return new Promise((resolve, reject) => {
			var fr = new FileReader();
			fr.onload = () => {
				resolve(fr.result )
			};
			fr.onerror = reject;
			fr.readAsArrayBuffer(file);
		});
	}

	function handleUpload(){
		console.log(files)
		const file = files[0];
		if (file === undefined) {
			// TODO: error handling
			return;
		}

		beatPath = window.URL.createObjectURL(file);

		readAsArrayBuffer(file)
			.then((buffer) => audioCtx.decodeAudioData(buffer))
			.then((audioBuffer) => guess(audioBuffer))
			.then((obj) => {
				console.log("done")
				finishProcessingSong(obj)
			})
			.catch((err) => console.error(err));
	}
</script>

<svelte:head>
	<title>beats me</title>
	<meta name="description" content="beats me - a quick demo" />
</svelte:head>

<section>
	<h1>beats me</h1>
	{#if loading || !initialized}
		<h2>Loading...</h2>
	{:else if !loaded}
		<button on:click={setup}> Start </button>
	{:else}
		<h2>
			BPM: {bpm.toFixed(2)} | Offset: {offset.toFixed(2)}s | Period: {period.toFixed(2)}s
		</h2>
		<audio on:play={play} on:pause={pause} bind:this={audioElement} src={beatPath} controls>
			<track kind="captions" />
		</audio>
		<fieldset>
			<input type="file" id="audio-file" accept="audio/mpeg, audio/ogg, audio/*" bind:files />
			<button type="button" id="upload" style="margin-top: 20px;" on:click={handleUpload}>Upload</button>
		</fieldset>
		<p class="big-text">
			{#if showBeat}
				<span style="color:green">on</span>
			{:else}
				off
			{/if}
		</p>
	{/if}
</section>

<style>
	section {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		flex: 0.6;
	}

	h1 {
		width: 100%;
	}

	.big-text {
		font-size: 10vw;
	}
</style>
