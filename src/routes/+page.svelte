<script>
	import { onMount } from 'svelte';
	import { guess } from 'web-audio-beat-detector';

	const beatPath = '/qwerhacks.mp3';

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
	let showBeat = false;

	onMount(init);

	function init() {
		initialized = true;
	}

	function setup() {
		loading = true;
		if (!audioCtx) {
			const AudioContext = window.AudioContext;
			audioCtx = new AudioContext();
		}
		fetch(beatPath)
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
