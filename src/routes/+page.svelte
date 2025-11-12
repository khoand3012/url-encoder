<script lang="ts">
	let url = $state('')

	let params = $state([
		{
			key: '',
			value: ''
		}
	])

	let result = $state('')

	let copyLabel = $state('Copy')

	const handleChangeKey = (e: Event, index: number) => {
		const newKey = (e.target as HTMLInputElement)?.value
		if (newKey) params[index].key = newKey
	}

	const handleChangeValue = (e: Event, index: number) => {
		const newValue = (e.target as HTMLTextAreaElement)?.value
		if (newValue) params[index].value = newValue
	}

	const addParam = () => {
		params.push({
			key: '',
			value: ''
		})
	}

	const removeParam = (index: number) => {
		if (params.length === 1) return
		params.splice(index, 1)
	}

	const generateURL = () => {
		let resultUrl = url
		params.forEach((param, index) => {
			if (index === 0) {
				resultUrl += '?'
			} else {
				resultUrl += '&'
			}
			resultUrl += `${param.key}=${encodeURIComponent(param.value.trim())}`
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
</script>

<div class="mx-20 my-20">
	<h1 class="mx-auto mb-10 text-3xl font-bold">URL encoding generator</h1>
	<div class="mx-auto w-full max-w-2xl">
		<div class="mb-4 rounded bg-white px-8 pt-6 pb-8 shadow-md">
			<div class="mb-4">
				<label class="mb-2 block text-sm font-bold text-gray-700" for="url">Url</label>
				<input
					class="focus:shadow-outline w-full appearance-none rounded border px-3 py-2 leading-tight text-gray-700 shadow focus:outline-none"
					type="text"
					name="url"
					id="url"
					bind:value={url}
				/>
			</div>
			{#each params as param, index}
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
					<div class="col-span-8">
						<label class="mb-2 block text-sm font-bold text-gray-700" for={`value-${index}`}
							>Value</label
						>
						<textarea
							class="focus:shadow-outline w-full appearance-none rounded border px-3 py-2 leading-tight text-gray-700 shadow focus:outline-none"
							name={`value-${index}`}
							id={`value-${index}`}
							bind:value={param.value}
							oninput={(e) => handleChangeValue(e, index)}
						></textarea>
					</div>
					<button
						onclick={() => removeParam(index)}
						tabindex="-1"
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
					class="col-span-10 appearance-none rounded border text-gray-700 shadow"
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
