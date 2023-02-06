<script lang="ts">
    import { createEventDispatcher, tick } from "svelte";
    import { fly } from "svelte/transition";
    import { BlockTitle } from "@gradio/atoms";

    export let value: string = "";
	export let lines: number = 1;
    export let choices: Array<string>;
    export let delimiter: string;
    export let label: string;
    export let show_label: boolean;
	export let max_lines: number | false;
    export let disabled = false;

    let el: HTMLTextAreaElement | HTMLInputElement;
    const multiselect = delimiter !== undefined;

	$: value, el && lines !== max_lines && resize({ target: el });
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

    function getTermRange(selectionStart: number) {
        const termStart = ((selectionStart <= 0 || !multiselect) ? 0
            : (value.lastIndexOf(delimiter, selectionStart - 1) + 1)
        );
        const termEnd = (!multiselect ? value.length
            : (value.indexOf(delimiter, selectionStart) == -1 ? value.length
                : value.indexOf(delimiter, selectionStart))
        );
        return [termStart, termEnd];
    }

    async function handleInput(e: any) {
        await tick();
        const { selectionStart } = this;
        const [termStart, termEnd] = getTermRange(selectionStart);
        searchTerm = value.slice(termStart, termEnd);
    }

    async function handleOptionMousedown(e: any) {
        await tick();
        const option = e.target.dataset.value;
        searchTerm = "";
        if (option !== undefined) {
            // replace input until the cursor
            const { selectionStart } = el;
            const [termStart, termEnd] = getTermRange(selectionStart || 0);
            if (multiselect) {
                value = value.slice(0, termStart) + option + delimiter + (
                    value[termEnd] === "," ? value.slice(termEnd+1) : value.slice(termEnd));
            } else {
                value = option;
                el.blur();
            }
        }
    }

	async function resize(
		event: Event | { target: HTMLTextAreaElement | HTMLInputElement }
	) {
		await tick();
		if (lines === max_lines) return;

		let max =
			max_lines === false
				? false
				: max_lines === undefined // default
				? 21 * 11
				: 21 * (max_lines + 1);
		let min = 21 * (lines + 1);

		const target = event.target as HTMLTextAreaElement;
		target.style.height = "1px";

		let scroll_height;
		if (max && target.scrollHeight > max) {
			scroll_height = max;
		} else if (target.scrollHeight < min) {
			scroll_height = min;
		} else {
			scroll_height = target.scrollHeight;
		}

		target.style.height = `${scroll_height}px`;
	}

	function text_area_resize(el: HTMLTextAreaElement, value: string) {
		if (lines === max_lines) return;
		el.style.overflowY = "scroll";
		el.addEventListener("input", resize);

		if (!value.trim()) return;
		resize({ target: el });

		return {
			destroy: () => el.removeEventListener("input", resize)
		};
	}
</script>

<!-- svelte-ignore a11y-label-has-associated-control -->
<label>
    <BlockTitle {show_label}>{label}</BlockTitle>

    <textarea
        data-testid="textbox"
        use:text_area_resize={value}
        class="scroll-hide"
        bind:value
        bind:this={el}
        rows={lines}
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
	label {
		display: block;
		width: 100%;
	}

	textarea {
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

	textarea:focus {
		--ring-color: var(--color-focus-ring);
		border-color: var(--input-border-color-focus);
	}

	textarea[disabled] {
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
