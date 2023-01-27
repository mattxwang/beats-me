<script>
	/**
	 * @type {any[]}
	 */
	export let options = [];
	export let display_func = (/** @type {any} */ a) => a;
	/**
	 * @type {any}
	 */
	export let value;
</script>

<select bind:value on:change>
	{#each options as option}
		<option value={option}>{display_func(option)}</option>
	{/each}
</select>
