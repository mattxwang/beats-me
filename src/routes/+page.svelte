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
	<h1 class="text-5xl my-6">beats me</h1>
	{#if loading || !initialized}
		<h2>Loading...</h2>
	{:else if !loaded}
		<button class="btn btn-blue" on:click={setup}> start </button>
	{:else}
		<p class="my-3 py-3 text-white">
			{#if showBeat}
				<div
					class="w-36 h-36 bg-green-700 rounded-full flex justify-center items-center text-center p-5 shadow-xl"
				>
					on
				</div>
			{:else}
				<!-- off -->
				<div
					class="w-36 h-36 bg-gray-900 rounded-full flex justify-center items-center text-center p-5 shadow-xl"
				>
					off
				</div>
			{/if}
		</p>
		<div class="max-w-sm rounded overflow-hidden shadow-lg px-6 py-4 bg-white">
			<dl class="sm:grid sm:grid-cols-2 sm:gap-4 sm:px-6">
				<dt class="text-gray-500">Song</dt>
				<dd class="text-gray-900">{beatPath}</dd>

				<dt class="text-gray-500">BPM / Period</dt>
				<dd class="text-gray-900">{bpm.toFixed(2)} / {period.toFixed(2)}s</dd>

				<dt class="text-gray-500">Offset</dt>
				<dd class="text-gray-900">{offset.toFixed(2)}s</dd>
			</dl>
			<audio class="mt-3" on:play={play} on:pause={pause} src={beatPath} controls>
				<track kind="captions" />
			</audio>
		</div>

		<p class="my-3">choose from our presets:</p>
		<Select
			options={PRESETS}
			display_func={(o) => o}
			bind:value={selectedBeat}
			on:change={handleSelectChange}
		/>
		<p class="my-3">or, upload a custom song</p>
		<label
			class="w-64 flex flex-col items-center px-4 py-6 bg-white text-blue-500 rounded-lg shadow-lg tracking-wide border border-blue-500 cursor-pointer hover:bg-blue-500 hover:text-white"
		>
			<svg
				class="w-8 h-8"
				fill="currentColor"
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 20 20"
				aria-hidden="true"
			>
				<path
					d="M16.88 9.1A4 4 0 0 1 16 17H5a5 5 0 0 1-1-9.9V7a3 3 0 0 1 4.52-2.59A4.98 4.98 0 0 1 17 8c0 .38-.04.74-.12 1.1zM11 11h3l-4-4-4 4h3v3h2v-3z"
				/>
			</svg>
			<span class="mt-2 text-base leading-normal">select a file</span>
			<input
				class="hidden"
				type="file"
				id="audio-file"
				accept="audio/mpeg, audio/ogg, audio/*"
				bind:files
				on:change={handleUpload}
			/>
		</label>
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
</style>
