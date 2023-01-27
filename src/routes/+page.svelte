<script>
	import { onMount } from 'svelte';
	import Select from './Select.svelte';
	import { guess } from 'web-audio-beat-detector';

	const DEFAULT_BEAT_PATH = '/qwerhacks.mp3';

	const PRESETS = [
		DEFAULT_BEAT_PATH,
		'/champagne.mp3',
		'/class.mp3',
		'/delight.mp3',
		'/drawl.mp3',
		'/haze.mp3',
		'/heaven.mp3',
		'/inferno.mp3',
		'/lounge.mp3',
		'/monster.mp3',
		'/tennis.mp3'
	];

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
	/**
	 * @type {NodeJS.Timer | undefined}
	 */
	let beatInterval;
	let beatPath = DEFAULT_BEAT_PATH;
	let selectedBeat = DEFAULT_BEAT_PATH;
	let showBeat = false;

	/**
	 * @type {FileList}
	 */
	let files;

	onMount(() => (initialized = true));

	/**
	 * @param {{}} _
	 * @param {string} file
	 */
	function setup(_, file = DEFAULT_BEAT_PATH) {
		loading = true;
		beatPath = file;
		if (!audioCtx) {
			const AudioContext = window.AudioContext;
			audioCtx = new AudioContext();
		}
		fetch(file)
			.then((data) => data.arrayBuffer())
			.then((buffer) => audioCtx.decodeAudioData(buffer))
			.then((audioBuffer) => guess(audioBuffer))
			.then((obj) => {
				bpm = obj.bpm;
				offset = obj.offset;
				// type error with TS :(
				// tempo = obj.tempo;
				period = 1 / (obj.bpm / 60);
				loaded = true;
				loading = false;
			})
			.catch((err) => console.error(err));
	}

	/**
	 * @param {Event} e
	 */
	function handleSelectChange(e) {
		if (!e.target) {
			// TODO: errors?
			return;
		}
		// TODO: fix this tsignore?
		/**
		 * @type {HTMLSelectElement}
		 */
		// @ts-ignore
		const target = e.target;
		selectedBeat = target.value;
		setup({}, selectedBeat);
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

	function handleUpload() {
		const file = files[0];
		if (file === undefined) {
			// TODO: error handling
			return;
		}

		const path = window.URL.createObjectURL(file);
		setup({}, path);
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
			{beatPath} | BPM: {bpm.toFixed(2)} | Offset: {offset.toFixed(2)}s | Period: {period.toFixed(
				2
			)}s
		</h2>
		<audio on:play={play} on:pause={pause} src={beatPath} controls>
			<track kind="captions" />
		</audio>
		<p>choose from our presets:</p>
		<Select
			options={PRESETS}
			display_func={(o) => o}
			bind:value={selectedBeat}
			on:change={handleSelectChange}
		/>
		<p>or, upload a custom song</p>
		<fieldset>
			<input type="file" id="audio-file" accept="audio/mpeg, audio/ogg, audio/*" bind:files />
			<button type="button" id="upload" style="margin-top: 20px;" on:click={handleUpload}
				>Upload</button
			>
		</fieldset>
		<p class="big-text">
			{#if showBeat}
				<span class="text-green">on</span>
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

	.text-green {
		color: green;
	}
</style>
