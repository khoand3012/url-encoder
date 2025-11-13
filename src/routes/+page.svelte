<script lang="ts">
	import { EditorView, basicSetup } from 'codemirror'
	import { json } from '@codemirror/lang-json'
	import { EditorState } from '@codemirror/state'

	type Param = {
		key: string
		value: string
		editor: EditorView | null
		editorDiv: HTMLElement | null
	}

	let baseUrl = $state('')

	let params: Param[] = $state([
		{
			key: '',
			value: '',
			editor: null,
			editorDiv: null
		}
	])

	let result = $state('')
	let copyLabel = $state('Copy')
	let toastMessage = $state('')
	let orgUrl = $state('')

	const createEditor = (container: HTMLElement, initialValue: string, index: number) => {
		const state = EditorState.create({
			doc: initialValue,
			extensions: [
				basicSetup,
				json(),
				EditorView.updateListener.of((update) => {
					if (update.docChanged) {
						params[index].value = update.state.doc.toString()
					}
				})
			]
		})

		return new EditorView({
			state,
			parent: container
		})
	}

	const handleChangeKey = (e: Event, index: number) => {
		const newKey = (e.target as HTMLInputElement)?.value
		if (newKey !== undefined) params[index].key = newKey
	}

	const addParam = () => {
		params.push({
			key: '',
			value: '',
			editor: null,
			editorDiv: null
		})
	}

	const removeParam = (index: number) => {
		if (params.length === 1) return

		// Destroy the editor before removing
		if (params[index].editor) {
			params[index].editor.destroy()
		}

		params.splice(index, 1)
	}

	const formatJSON = (index: number) => {
		const editor = params[index].editor
		if (!editor) return

		const currentValue = editor.state.doc.toString()

		try {
			const parsed = JSON.parse(currentValue)
			const formatted = JSON.stringify(parsed, null, 2)

			editor.dispatch({
				changes: {
					from: 0,
					to: editor.state.doc.length,
					insert: formatted
				}
			})
		} catch (e) {
			toastMessage = e instanceof Error ? e.message : 'Invalid JSON format'
			setTimeout(() => {
				toastMessage = ''
			}, 3000)
		}
	}

	const generateURL = () => {
		let resultUrl = baseUrl
		params.forEach((param, index) => {
			if (index === 0) {
				resultUrl += '?'
			} else {
				resultUrl += '&'
			}
			let value = ''
			try {
				value = JSON.stringify(JSON.parse(param.value))
			} catch {
				value = param.value.trim()
			}
			resultUrl += `${param.key}=${encodeURIComponent(value)}`
		})
		result = resultUrl
	}

	const copyText = () => {
		navigator.clipboard.writeText(result)
		copyLabel = 'Copied'
		setTimeout(() => {
			copyLabel = 'Copy'
		}, 3000)
	}

	const extractURL = () => {
		if (!orgUrl) return
		const parsedUrl = new URL(orgUrl)
		const parsedParams: Param[] = []

		baseUrl = parsedUrl.href.split('?')[0]

		parsedUrl.searchParams.forEach((value, key) => {
			parsedParams.push({
				key,
				value,
				editor: null,
				editorDiv: null
			})
		})

		if (params.length > 0) {
			params.forEach((param) => {
				if (param.editor) param.editor.destroy()
			})
		}

		params = parsedParams

		orgUrl = ''
	}

	// Initialize editors when divs are mounted
	$effect(() => {
		params.forEach((param, index) => {
			if (param.editorDiv && !param.editor) {
				param.editor = createEditor(param.editorDiv, param.value, index)
			}
		})
	})
</script>

<div class="mx-20 my-20">
	{#if toastMessage !== ''}
		<div class="animate-slide-in fixed top-4 right-4 z-50">
			<div class="rounded-lg bg-red-500 px-6 py-4 text-white shadow-lg">
				<div class="flex items-center gap-3">
					<svg class="h-6 w-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"
						/>
					</svg>
					<span class="font-medium">{toastMessage}</span>
				</div>
			</div>
		</div>
	{/if}
	<h1 class="mx-auto mb-10 text-3xl font-bold">URL encoding generator</h1>
	<div class="mx-auto w-full max-w-2xl">
		<div class="mb-4 rounded bg-white px-8 pt-6 pb-8 shadow-md">
			<label for="result" class="mb-2 block text-sm font-bold text-gray-700">URL to extract:</label>
			<div class="grid grid-cols-12 gap-2">
				<input
					class="col-span-9 appearance-none rounded border px-3 py-2 text-gray-700 shadow"
					type="text"
					name="result"
					id="result"
					bind:value={orgUrl}
				/>
				<button
					tabindex="-1"
					onclick={extractURL}
					class={`focus:shadow-outline col-span-3 rounded bg-green-500 px-4 py-2 font-bold text-white hover:bg-green-700 focus:outline-none`}
					>Extract URL</button
				>
			</div>
		</div>
		<div class="mb-4 rounded bg-white px-8 pt-6 pb-8 shadow-md">
			<div class="mb-4">
				<label class="mb-2 block text-sm font-bold text-gray-700" for="url">Base Url</label>
				<input
					class="focus:shadow-outline w-full appearance-none rounded border px-3 py-2 leading-tight text-gray-700 shadow focus:outline-none"
					type="text"
					name="url"
					id="url"
					bind:value={baseUrl}
				/>
			</div>
			{#each params as param, index (index)}
				<div class="mb-4 grid grid-cols-12 gap-2">
					<div class="col-span-3">
						<label class="mb-2 block text-sm font-bold text-gray-700" for={`key-${index}`}
							>Key</label
						>
						<input
							class="focus:shadow-outline w-full appearance-none rounded border px-3 py-2 leading-tight text-gray-700 shadow focus:outline-none"
							type="text"
							name={`key-${index}`}
							id={`key-${index}`}
							value={param.key}
							onchange={(e) => handleChangeKey(e, index)}
						/>
					</div>
					<div class="col-span-7">
						<label class="mb-2 block text-sm font-bold text-gray-700" for={`value-${index}`}
							>Value</label
						>
						<div
							bind:this={param.editorDiv}
							class="focus:shadow-outline rounded border shadow"
						></div>
					</div>
					<button
						onclick={() => formatJSON(index)}
						tabindex="-1"
						title="Format JSON"
						class="focus:shadow-outline col-span-1 mt-7 aspect-square rounded bg-purple-500 px-4 py-2 font-bold text-white hover:bg-purple-700 focus:outline-none"
						>{'{}'}</button
					>
					<button
						onclick={() => removeParam(index)}
						tabindex="-1"
						title="Remove Param"
						class="focus:shadow-outline col-span-1 mt-7 aspect-square rounded bg-red-500 px-4 py-2 font-bold text-white hover:bg-red-700 focus:outline-none"
						>-</button
					>
				</div>
			{/each}
			<button
				tabindex="-1"
				onclick={addParam}
				class="focus:shadow-outline col-span-1 mt-7 rounded bg-blue-500 px-4 py-2 font-bold text-white hover:bg-blue-700 focus:outline-none"
				>Add Param</button
			>
			<button
				tabindex="-1"
				onclick={generateURL}
				class="focus:shadow-outline col-span-1 mt-7 rounded bg-green-500 px-4 py-2 font-bold text-white hover:bg-green-700 focus:outline-none"
				>Generate URL</button
			>
		</div>
		<div class="rounded bg-white px-8 pt-6 pb-8 shadow-md">
			<label for="result" class="mb-2 block text-sm font-bold text-gray-700">Result URL:</label>
			<div class="grid grid-cols-12 gap-2">
				<input
					class="col-span-10 appearance-none rounded border px-3 py-2 text-gray-700 shadow"
					type="text"
					name="result"
					id="result"
					value={result}
					disabled
				/>
				<button
					tabindex="-1"
					onclick={copyText}
					class={`focus:shadow-outline col-span-2 rounded px-4 py-2 font-bold text-white hover:bg-green-700 focus:outline-none ${copyLabel === 'Copy' ? 'bg-green-500' : 'bg-green-700'}`}
					>{copyLabel}</button
				>
			</div>
		</div>
	</div>
</div>

<style>
	:global(.cm-editor) {
		min-height: 40px;
		max-height: 300px;
		overflow: auto;
	}

	:global(.cm-scroller) {
		overflow: auto;
	}

	@keyframes slide-in {
		from {
			transform: translateX(100%);
			opacity: 0;
		}
		to {
			transform: translateX(0);
			opacity: 1;
		}
	}

	.animate-slide-in {
		animation: slide-in 0.3s ease-out;
	}
</style>
