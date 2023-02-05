<script lang="ts">
    import { createEventDispatcher, tick } from "svelte";
    import { fly } from "svelte/transition";
    import { BlockTitle } from "@gradio/atoms";

    export let value: string = "";
    export let choices: Array<string>;
    export let delimiter: string;
    export let label: string;
    export let show_label: boolean;
    export let disabled = false;

    let el: HTMLTextAreaElement | HTMLInputElement;
    const multiselect = delimiter !== undefined;

    $: handle_change(value);

    const dispatch = createEventDispatcher<{
        change: String;
    }>();

    function handle_change(val: string) {
        dispatch("change", val);
    }

    function matcher(query: string, candidate: string) {
        return (query.trim()
            ? candidate.toLowerCase().includes(query.toLowerCase())
            : candidate);
    }

    let searchTerm: string = "",
        showOptions = false;

    $: filtered = choices.filter((opt) => matcher(searchTerm, opt));
    $: console.log(searchTerm);

    function optionsVisibility(show: boolean) {
        if (typeof show === "boolean") {
            showOptions = show;
        } else {
            showOptions = !showOptions;
        }
    }

    function getTermStart(termEnd: number) {
        if (termEnd <= 0) {
            return 0;
        }
        if (multiselect) {
            return value.lastIndexOf(delimiter, termEnd - 1) + 1;
        } else {
            return 0;
        }
    }

    async function handleInput(e: any) {
        await tick();
        const { selectionStart } = this;
        const termEnd = selectionStart;
        const termStart = getTermStart(termEnd);
        searchTerm = value.slice(termStart, termEnd);
    }

    async function handleOptionMousedown(e: any) {
        await tick();
        const option = e.target.dataset.value;
        searchTerm = "";
        if (option !== undefined) {
            // replace input until the cursor
            const { selectionStart } = el;
            const termStart = getTermStart(selectionStart || 0);
            console.log(termStart, selectionStart);
            if (multiselect) {
                value = value.slice(0, termStart) + option + delimiter + value.slice(selectionStart);
            } else {
                value = option;
                el.blur();
            }
        }
    }
</script>

<!-- svelte-ignore a11y-label-has-associated-control -->
<label>
    <BlockTitle {show_label}>{label}</BlockTitle>

    <input
        data-testid="textbox"
        type="text"
        class="scroll-hide"
        bind:value
        bind:this={el}
        {disabled}
        on:input={handleInput}
        on:focus={() => optionsVisibility(true)}
        on:blur={() => optionsVisibility(false)}
    />

    {#if showOptions && !disabled}
        <ul
            class="options"
            transition:fly={{ duration: 200, y: 5 }}
            on:mousedown|preventDefault={handleOptionMousedown}
        >
            {#each filtered as choice}
                <li
                    class="item"
                    data-value={choice}
                >
                    {choice}
                </li>
            {/each}
        </ul>
    {/if}
</label>

<style>
	input {
		--ring-color: transparent;
		display: block;
		position: relative;
		outline: none !important;
		box-shadow: 0 0 0 var(--shadow-spread) var(--ring-color),
			var(--shadow-inset);
		border: 1px solid var(--input-border-color-base);
		border-radius: var(--radius-lg);
		background: var(--input-background-base);
		padding: var(--size-2-5);
		width: 100%;
		color: var(--color-text-body);
		font-size: var(--scale-00);
		line-height: var(--line-sm);
	}

	input:focus {
		--ring-color: var(--color-focus-ring);
		border-color: var(--input-border-color-focus);
	}

	input[disabled] {
		box-shadow: none;
	}

	.options {
		position: absolute;
		z-index: var(--layer-5);
		margin-left: 0;
		box-shadow: var(--shadow-drop-lg);
		border-radius: var(--radius-lg);
		background: var(--color-background-primary);
		width: var(--size-full);
		max-height: var(--size-32);
		overflow: auto;
		color: var(--color-text-body);
		list-style: none;
	}

	.item {
		display: flex;
		cursor: pointer;
		padding: var(--size-2);
	}

	.item:hover {
		background: var(--color-background-secondary);
	}


</style>
