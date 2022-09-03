<script lang="ts">
	export let name: string;
	let classname = '';
	export { classname as class };
</script>

<div class={`${classname} tooltip`}>
	<slot />
	<span class="tooltiptext">{name}</span>
</div>

<style>
	/* Tooltip text */
	.tooltip .tooltiptext {
		visibility: hidden;
		width: max-content;
		background-color: black;
		color: #fff;
		text-align: center;
		padding: 4px 8px;
		border: 1px solid white;

		/* Position the tooltip text - see examples below! */
		position: absolute;
		z-index: 1;
	}

	/* Show the tooltip text when you mouse over the tooltip container */
	.tooltip:hover .tooltiptext {
		visibility: visible;
	}
</style>
