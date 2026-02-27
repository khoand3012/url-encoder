<script lang="ts">
	import type { EditorView } from 'codemirror'
	export type ParamType = {
		id: string
		key: string
		value: string
		editor: EditorView | null
		editorDiv: HTMLElement | null
	}

	export type ParamProps = {
		param: ParamType
		index: number
		onChangeKey: (e: Event, index: number) => void
		onFormatJSON: (index: number) => void
		onRemove: (index: number) => void
	}

	let { param, index, onChangeKey, onFormatJSON, onRemove }: ParamProps = $props()
</script>

<div class="mb-4 grid grid-cols-12 gap-2">
	<div class="col-span-3">
		<label class="mb-2 block text-sm font-bold text-gray-700" for={`key-${index}`}>Key</label>
		<input
			class="focus:shadow-outline w-full appearance-none rounded border px-3 py-2 leading-tight text-gray-700 shadow focus:outline-none"
			type="text"
			name={`key-${index}`}
			id={`key-${index}`}
			value={param.key}
			onchange={(e) => onChangeKey(e, index)}
		/>
	</div>
	<div class="col-span-7">
		<label class="mb-2 block text-sm font-bold text-gray-700" for={`value-${index}`}>Value</label>
		<div bind:this={param.editorDiv} class="focus:shadow-outline rounded border shadow"></div>
	</div>
	<button
		onclick={() => onFormatJSON(index)}
		tabindex="-1"
		title="Format JSON"
		class="focus:shadow-outline col-span-1 mt-7 aspect-square rounded bg-purple-500 px-4 py-2 font-bold text-white hover:bg-purple-700 focus:outline-none"
		>{'{}'}</button
	>
	<button
		onclick={() => onRemove(index)}
		tabindex="-1"
		title="Remove Param"
		class="focus:shadow-outline col-span-1 mt-7 aspect-square rounded bg-red-500 px-4 py-2 font-bold text-white hover:bg-red-700 focus:outline-none"
		>-</button
	>
</div>
