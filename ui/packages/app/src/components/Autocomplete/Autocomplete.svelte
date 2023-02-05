<script lang="ts">
    import { Autocomplete } from "@gradio/form";
    import { Block } from "@gradio/atoms";
	import StatusTracker from "../StatusTracker/StatusTracker.svelte";
	import type { LoadingStatus } from "../StatusTracker/types";
	import type { Styles } from "@gradio/utils";

	export let label: string = "Textbox";
	export let elem_id: string = "";
	export let visible: boolean = true;
	export let value: string = "";
    export let choices: Array<string>;
    export let delimiter: string;
	export let show_label: boolean;
    export let style: Styles = {};
    export let loading_status: LoadingStatus;

    export let mode: "static" | "dynamic";
</script>

<Block
    {visible}
    {elem_id}
    disable={typeof style.container === "boolean" && !style.container}
>
    {#if loading_status}
        <StatusTracker {...loading_status} />
    {/if}

    <Autocomplete
        bind:value
        {label}
        {show_label}
        {choices}
        {delimiter}
        on:change
        disabled={mode === "static"}
    />
</Block>
