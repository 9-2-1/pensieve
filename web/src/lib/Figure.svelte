<!-- Modal.svelte -->
<script>
	import { ScrollArea } from '$lib/components/ui/scroll-area';
	import CopyToClipboard from '$lib/components/CopyToClipboard.svelte';
	import OCRTable from './OCRTable.svelte';
	import { marked } from 'marked';
	import {
		ChevronLeft,
		ChevronRight,
		X,
		Hash,
		Library,
		Folder,
		FileClock,
		IndentIncrease
	} from '@lucide/svelte';
	import { translateAppName } from '$lib/utils';
	import LucideIcon from '$lib/components/LucideIcon.svelte';
	import { onMount } from 'svelte';

	/**
	 * @typedef {Object} Props
	 * @property {string} id
	 * @property {number} library_id
	 * @property {number} created_at
	 * @property {number} folder_id
	 * @property {any} image
	 * @property {string} video
	 * @property {string} filepath
	 * @property {string} title
	 * @property {any} app_name
	 * @property {Array<string>} [tags]
	 * @property {Array<{key: string, source: string, value: any}>} [metadata_entries]
	 * @property {any} onClose
	 * @property {any} onNext
	 * @property {any} onPrevious
	 */

	/** @type {Props} */
	let {
		id,
		library_id,
		created_at,
		folder_id,
		image,
		video,
		filepath,
		title,
		app_name,
		tags = [],
		metadata_entries = [],
		onClose,
		onNext,
		onPrevious
	} = $props();

	let showDetails = $state(false);

	onMount(() => {
		// 从 localStorage 读取状态
		const savedState = localStorage.getItem('figureShowDetails');
		showDetails = savedState ? JSON.parse(savedState) : false;
	});

	function toggleDetails() {
		showDetails = !showDetails;
		// 保存状态到 localStorage
		localStorage.setItem('figureShowDetails', JSON.stringify(showDetails));
	}

	/**
	 * @param {any} data
	 * @returns {boolean}
	 */
	function isValidOCRDataStructure(data) {
		if (!Array.isArray(data)) return false;

		for (const item of data) {
			if (
				!item.hasOwnProperty('dt_boxes') ||
				!item.hasOwnProperty('rec_txt') ||
				!item.hasOwnProperty('score')
			) {
				return false;
			}

			if (
				!Array.isArray(item.dt_boxes) ||
				typeof item.rec_txt !== 'string' ||
				typeof item.score !== 'number'
			) {
				return false;
			}
		}

		return true;
	}
	// Remove items with key "timestamp" or "sequence" and sort metadata_entries, placing "ocr_result" at the end
	let sortedMetadataEntries = $derived([...metadata_entries]
		.filter(
			(entry) =>
				entry.key !== 'timestamp' &&
				entry.key !== 'sequence' &&
				entry.key !== 'active_app' &&
				entry.key !== 'active_window'
		)
		.sort((a, b) => {
			if (a.key === 'ocr_result') return 1;
			if (b.key === 'ocr_result') return -1;
			return 0;
		}));
</script>

<div
	class="fixed inset-0 bg-gray-600 bg-opacity-50 h-full w-full z-40 flex items-center justify-center"
	id="my-modal"
>
	<div
		class="relative mx-auto border w-11/12 max-w-[95vw] h-[95vh] shadow-lg rounded-md bg-white group"
	>
		<div class="absolute inset-0 px-10 py-4">
			<!-- Button container -->
			<div class="group absolute inset-x-0 h-full">
				<button
					class="absolute p-2 left-2 top-1/2 transform -translate-y-1/2 rounded-full hover:bg-gray-100 bg-white/80 border opacity-0 group-hover:opacity-100 flex z-[51] transition-all duration-200"
					onclick={onPrevious}
				>
					<ChevronLeft size={24} class="text-indigo-600" />
				</button>
				<button
					class="absolute p-2 right-2 top-1/2 transform -translate-y-1/2 rounded-full hover:bg-gray-100 bg-white/80 border opacity-0 group-hover:opacity-100 flex z-[51] transition-all duration-200"
					onclick={onNext}
				>
					<ChevronRight size={24} class="text-indigo-600" />
				</button>
			</div>

			<div class="flex flex-col md:flex-row h-full relative">
				<!-- Image container -->
				<div class="flex-none {showDetails ? 'w-full md:w-1/2' : 'w-full'} flex flex-col h-full">
					<div class="mb-2 relative z-[52]">
						<div class="flex justify-between items-center">
							<div class="flex-1"></div>
							<div class="flex items-center text-lg leading-tight font-medium text-black {showDetails ? 'w-full' : 'w-4/5'}">
								<div class="flex items-center justify-between w-full">
									<div class="flex-1 flex items-center justify-center min-w-0">
										<div class="flex items-center space-x-2 min-w-0">
											<LucideIcon name={translateAppName(app_name) || 'Image'} size={24} />
											<p class="truncate">{title}</p>
											{#if !showDetails}
											<span class="inline-flex items-center text-sm text-gray-500 font-mono pl-4">
												<FileClock size={16} class="mr-1 text-gray-500" />
												{new Date(created_at).toLocaleString()}
											</span>
											{/if}
										</div>
									</div>
									<button
										class="p-2 hover:bg-gray-100 rounded-full transition-colors flex-shrink-0"
										onclick={toggleDetails}
									>
										<IndentIncrease 
											size={24} 
											class={showDetails ? 'text-indigo-600' : 'text-gray-400'} 
										/>
									</button>
								</div>
							</div>
							<div class="flex-1"></div>
						</div>
					</div>

					{#if showDetails}
						<div class="mb-2 mr-2 pb-2 border-b border-gray-300">
							<span class="mt-1 text-sm leading-tight font-medium text-gray-500 font-mono">
								<span class="inline-flex mr-4">
									<Library
										size={16}
										class="uppercase tracking-wide text-sm text-indigo-600 font-bold mr-1"
									/>
									{library_id}
								</span>

								<span class="inline-flex mr-4">
									<Folder
										size={16}
										class="uppercase tracking-wide text-sm text-indigo-600 font-bold mr-1"
									/>
									{folder_id}
								</span>

								<span class="inline-flex mr-4">
									<Hash
										size={16}
										class="uppercase tracking-wide text-sm text-indigo-600 font-bold mr-1"
									/>
									{id}
								</span>

								<span class="inline-flex mr-4">
									<FileClock
										size={16}
										class="uppercase tracking-wide text-sm text-indigo-600 font-bold mr-1 font-mono"
									/>
									{new Date(created_at).toLocaleString()}
								</span>
							</span>

							<div>
								<span class="mt-1 text-xs leading-tight font-xs text-gray-500 font-mono">
									{filepath}
								</span>
							</div>
						</div>
					{/if}

					<div class="relative flex-1 overflow-hidden flex items-center justify-center  {showDetails ? "mr-2" : ""}">
						<a href={video} target="_blank" rel="noopener noreferrer" class="block w-full h-full flex items-center justify-center">
							<img
								class="h-full object-contain rounded-lg drop-shadow-md"
								src={image}
								alt={title}
							/>
						</a>
					</div>
				</div>
				<!-- Description container -->
				{#if showDetails}
					<ScrollArea class="mt-4 md:mt-0 md:ml-6 md:w-1/2 h-full">
						{#if tags.length > 0}
							<div class="mb-4">
								<div class="uppercase tracking-wide text-sm text-indigo-600 font-bold">TAGS</div>
								<div class=" text-gray-600">
									{#each tags as tag}
										<span class="text-base text-gray-500 inline-block">{tag}</span>
									{/each}
								</div>
							</div>
						{/if}
						<div class="uppercase tracking-wide text-sm text-indigo-600 font-bold">METADATA</div>
						<div class="mt-2 text-gray-600">
							{#each sortedMetadataEntries as entry}
								<div class="mb-2">
									<span class="font-bold flex items-center">
										{entry.key}
										<CopyToClipboard text={entry.value} />
									</span>
									{#if typeof entry.value === 'object'}
										{#if isValidOCRDataStructure(entry.value)}
											<OCRTable ocrData={entry.value} />
										{:else}
											<pre class="bg-gray-100 p-2 rounded overflow-y-auto max-h-96">{JSON.stringify(
													entry.value,
													null,
													2
												)}</pre>
										{/if}
									{:else}
										<!-- Render markdown content -->
										<div class="prose">
											{@html marked(entry.value)}
										</div>
									{/if}
									<span class="text-sm text-gray-500">({entry.source})</span>
								</div>
							{/each}
						</div>
					</ScrollArea>
				{/if}
			</div>

			<div class="absolute top-2 right-2 z-[52]">
				<button
					class="p-2 rounded-full hover:bg-gray-100 bg-white/80 opacity-0 group-hover:opacity-100 transition-all duration-200"
					onclick={onClose}
				>
					<X size={24} class="text-indigo-600" />
				</button>
			</div>
		</div>
	</div>
</div>
