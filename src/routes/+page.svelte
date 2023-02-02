<script>
	import { onMount } from 'svelte';
	import { guess } from 'web-audio-beat-detector';
	import FileSelect from './FileSelect.svelte';
	import Radio from './Radio.svelte';
	import Select from './Select.svelte';

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

	const MULTIPLIER_OPTIONS = [
		{
			label: 'Normal',
			value: '1'
		},
		{
			label: 'Double-time',
			value: '2'
		},
		{
			label: 'Half-time',
			value: '0.5'
		},
		{
			label: 'Quad-time',
			value: '4'
		},
		{
			label: 'Quarter-time',
			value: '0.25'
		}
	];

	/**
	 * @type {AudioContext}
	 */
	let audioCtx;

	let bpm = 0,
		originalBpm = 0,
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

	/**
	 * @type {string}
	 */
	let selectedMultiplier;

	onMount(() => (initialized = true));

	/**
	 * @param {number} bpm
	 * @returns {number}
	 */
	function clampBPM(bpm) {
		if (bpm > 160) {
			return clampBPM(bpm / 2);
		}
		if (bpm < 40) {
			return clampBPM(bpm * 2);
		}
		return bpm;
	}

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
				const clampedBPM = clampBPM(obj.bpm);
				bpm = clampedBPM;
				originalBpm = clampedBPM;
				offset = obj.offset;
				// type error with TS :(
				// tempo = obj.tempo;
				period = 1 / (clampedBPM / 60);
				loaded = true;
				loading = false;

				selectedMultiplier = MULTIPLIER_OPTIONS[0].value;
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

	function play() {
		setTimeout(() => {
			beatInterval = setInterval(() => {
				showBeat = true;
				setTimeout(() => {
					showBeat = false;
				}, 200);
			}, period * 1000);
		}, offset * 1000);
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

	function handleMultiplierChange() {
		const multpliedBpm = originalBpm * Number(selectedMultiplier);
		bpm = multpliedBpm;
		period = 1 / (multpliedBpm / 60);
		console.log(bpm);
	}
</script>

<svelte:head>
	<title>beats me</title>
	<meta name="description" content="beats me - a quick demo of client-side beat detection" />
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
				<div class="large-circle bg-green-700">on</div>
			{:else}
				<div class="large-circle bg-gray-900">off</div>
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
			<dl
				class="sm:grid sm:grid-cols-2 sm:gap-4 sm:px-6 py-4"
				role="radiogroup"
				aria-labelledby="multiplier-label"
			>
				<dt class="text-gray-500" id="multiplier-label">Multiplier</dt>
				<dd class="text-gray-900">
					<Radio
						options={MULTIPLIER_OPTIONS}
						bind:userSelected={selectedMultiplier}
						on:change={handleMultiplierChange}
					/>
				</dd>
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
		<FileSelect bind:files on:change={handleUpload} />
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
