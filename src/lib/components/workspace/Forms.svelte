<script lang="ts">
	import { toast } from 'svelte-sonner';
	import fileSaver from 'file-saver';
	const { saveAs } = fileSaver;

	import { onMount, getContext } from 'svelte';
	import { WEBUI_NAME, forms } from '$lib/stores';
	import { createNewForm, deleteFormByCommand, getForms } from '$lib/apis/forms';
	import { error } from '@sveltejs/kit';
	import { goto } from '$app/navigation';
	import FormMenu from './Forms/FormMenu.svelte';
	import EllipsisHorizontal from '../icons/EllipsisHorizontal.svelte';
	import DeleteConfirmDialog from '$lib/components/common/ConfirmDialog.svelte';

	const i18n = getContext('i18n');

	let importFiles = '';
	let query = '';
	let formsImportInputElement: HTMLInputElement;

	let showDeleteConfirm = false;
	let deleteForm = null;

	const shareHandler = async (form) => {
		toast.success($i18n.t('Redirecting you to OpenWebUI Community'));

		const url = 'https://openwebui.com';

		const tab = await window.open(`${url}/forms/create`, '_blank');
		window.addEventListener(
			'message',
			(event) => {
				if (event.origin !== url) return;
				if (event.data === 'loaded') {
					tab.postMessage(JSON.stringify(form), '*');
				}
			},
			false
		);
	};

	const cloneHandler = async (form) => {
		sessionStorage.form = JSON.stringify(form);
		goto('/workspace/forms/create');
	};

	const exportHandler = async (form) => {
		let blob = new Blob([JSON.stringify([form])], {
			type: 'application/json'
		});
		saveAs(blob, `form-export-${Date.now()}.json`);
	};

	const deleteHandler = async (form) => {
		const command = form.command;
		await deleteFormByCommand(localStorage.token, command);
		await forms.set(await getForms(localStorage.token));
	};
</script>

<svelte:head>
	<title>
		{$i18n.t('Forms')} | {$WEBUI_NAME}
	</title>
</svelte:head>

<div class="mb-3 flex justify-between items-center">
	<div class=" text-lg font-semibold self-center">{$i18n.t('Forms')}</div>
</div>

<div class=" flex w-full space-x-2">
	<div class="flex flex-1">
		<div class=" self-center ml-1 mr-3">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 20 20"
				fill="currentColor"
				class="w-4 h-4"
			>
				<path
					fill-rule="evenodd"
					d="M9 3.5a5.5 5.5 0 100 11 5.5 5.5 0 000-11zM2 9a7 7 0 1112.452 4.391l3.328 3.329a.75.75 0 11-1.06 1.06l-3.329-3.328A7 7 0 012 9z"
					clip-rule="evenodd"
				/>
			</svg>
		</div>
		<input
			class=" w-full text-sm pr-4 py-1 rounded-r-xl outline-none bg-transparent"
			bind:value={query}
			placeholder={$i18n.t('Search Forms')}
		/>
	</div>

	<div>
		<a
			class=" px-2 py-2 rounded-xl border border-gray-200 dark:border-gray-600 dark:border-0 hover:bg-gray-100 dark:bg-gray-800 dark:hover:bg-gray-700 transition font-medium text-sm flex items-center space-x-1"
			href="/workspace/forms/create"
		>
			<svg
				xmlns="http://www.w3.org/2000/svg"
				viewBox="0 0 16 16"
				fill="currentColor"
				class="w-4 h-4"
			>
				<path
					d="M8.75 3.75a.75.75 0 0 0-1.5 0v3.5h-3.5a.75.75 0 0 0 0 1.5h3.5v3.5a.75.75 0 0 0 1.5 0v-3.5h3.5a.75.75 0 0 0 0-1.5h-3.5v-3.5Z"
				/>
			</svg>
		</a>
	</div>
</div>
<hr class=" dark:border-gray-850 my-2.5" />

<div class="my-3 mb-5">
	{#each $forms.filter((p) => query === '' || p.command.includes(query)) as form}
		<div
			class=" flex space-x-4 cursor-pointer w-full px-3 py-2 dark:hover:bg-white/5 hover:bg-black/5 rounded-xl"
		>
			<div class=" flex flex-1 space-x-4 cursor-pointer w-full">
				<a href={`/workspace/forms/edit?command=${encodeURIComponent(form.command)}`}>
					<div class=" flex-1 self-center pl-5">
						<div class=" font-semibold line-clamp-1">{form.command}</div>
						<div class=" text-xs overflow-hidden text-ellipsis line-clamp-1">
							{form.title}
						</div>
					</div>
				</a>
			</div>
			<div class="flex flex-row gap-0.5 self-center">
				<a
					class="self-center w-fit text-sm px-2 py-2 dark:text-gray-300 dark:hover:text-white hover:bg-black/5 dark:hover:bg-white/5 rounded-xl"
					type="button"
					href={`/workspace/forms/edit?command=${encodeURIComponent(form.command)}`}
				>
					<svg
						xmlns="http://www.w3.org/2000/svg"
						fill="none"
						viewBox="0 0 24 24"
						stroke-width="1.5"
						stroke="currentColor"
						class="w-4 h-4"
					>
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							d="M16.862 4.487l1.687-1.688a1.875 1.875 0 112.652 2.652L6.832 19.82a4.5 4.5 0 01-1.897 1.13l-2.685.8.8-2.685a4.5 4.5 0 011.13-1.897L16.863 4.487zm0 0L19.5 7.125"
						/>
					</svg>
				</a>

				<FormMenu
					shareHandler={() => {
						shareHandler(form);
					}}
					cloneHandler={() => {
						cloneHandler(form);
					}}
					exportHandler={() => {
						exportHandler(form);
					}}
					deleteHandler={async () => {
						deleteForm = form;
						showDeleteConfirm = true;
					}}
					onClose={() => {}}
				>
					<button
						class="self-center w-fit text-sm p-1.5 dark:text-gray-300 dark:hover:text-white hover:bg-black/5 dark:hover:bg-white/5 rounded-xl"
						type="button"
					>
						<EllipsisHorizontal className="size-5" />
					</button>
				</FormMenu>
			</div>
		</div>
	{/each}
</div>

<div class=" flex justify-end w-full mb-3">
	<div class="flex space-x-2">
		<input
			id="forms-import-input"
			bind:this={formsImportInputElement}
			bind:files={importFiles}
			type="file"
			accept=".json"
			hidden
			on:change={() => {
				console.log(importFiles);

				const reader = new FileReader();
				reader.onload = async (event) => {
					const savedForms = JSON.parse(event.target.result);
					console.log(savedForms);

					for (const form of savedForms) {
						await createNewForm(
							localStorage.token,
							form.command.charAt(0) === '/' ? form.command.slice(1) : form.command,
							form.title,
							form.content
						).catch((error) => {
							toast.error(error);
							return null;
						});
					}

					await forms.set(await getForms(localStorage.token));
				};

				reader.readAsText(importFiles[0]);
			}}
		/>

		<button
			class="flex text-xs items-center space-x-1 px-3 py-1.5 rounded-xl bg-gray-50 hover:bg-gray-100 dark:bg-gray-800 dark:hover:bg-gray-700 dark:text-gray-200 transition"
			on:click={() => {
				formsImportInputElement.click();
			}}
		>
			<div class=" self-center mr-2 font-medium line-clamp-1">{$i18n.t('Import Forms')}</div>

			<div class=" self-center">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					viewBox="0 0 16 16"
					fill="currentColor"
					class="w-4 h-4"
				>
					<path
						fill-rule="evenodd"
						d="M4 2a1.5 1.5 0 0 0-1.5 1.5v9A1.5 1.5 0 0 0 4 14h8a1.5 1.5 0 0 0 1.5-1.5V6.621a1.5 1.5 0 0 0-.44-1.06L9.94 2.439A1.5 1.5 0 0 0 8.878 2H4Zm4 9.5a.75.75 0 0 1-.75-.75V8.06l-.72.72a.75.75 0 0 1-1.06-1.06l2-2a.75.75 0 0 1 1.06 0l2 2a.75.75 0 1 1-1.06 1.06l-.72-.72v2.69a.75.75 0 0 1-.75.75Z"
						clip-rule="evenodd"
					/>
				</svg>
			</div>
		</button>

		<button
			class="flex text-xs items-center space-x-1 px-3 py-1.5 rounded-xl bg-gray-50 hover:bg-gray-100 dark:bg-gray-800 dark:hover:bg-gray-700 dark:text-gray-200 transition"
			on:click={async () => {
				// formsImportInputElement.click();
				let blob = new Blob([JSON.stringify($forms)], {
					type: 'application/json'
				});
				saveAs(blob, `forms-export-${Date.now()}.json`);
			}}
		>
			<div class=" self-center mr-2 font-medium line-clamp-1">{$i18n.t('Export Forms')}</div>

			<div class=" self-center">
				<svg
					xmlns="http://www.w3.org/2000/svg"
					viewBox="0 0 16 16"
					fill="currentColor"
					class="w-4 h-4"
				>
					<path
						fill-rule="evenodd"
						d="M4 2a1.5 1.5 0 0 0-1.5 1.5v9A1.5 1.5 0 0 0 4 14h8a1.5 1.5 0 0 0 1.5-1.5V6.621a1.5 1.5 0 0 0-.44-1.06L9.94 2.439A1.5 1.5 0 0 0 8.878 2H4Zm4 3.5a.75.75 0 0 1 .75.75v2.69l.72-.72a.75.75 0 1 1 1.06 1.06l-2 2a.75.75 0 0 1-1.06 0l-2-2a.75.75 0 0 1 1.06-1.06l.72.72V6.25A.75.75 0 0 1 8 5.5Z"
						clip-rule="evenodd"
					/>
				</svg>
			</div>
		</button>

		<!-- <button
						on:click={() => {
							loadDefaultForms();
						}}
					>
						dd
					</button> -->
	</div>
</div>

<div class=" my-16">
	<div class=" text-lg font-semibold mb-3 line-clamp-1">
		{$i18n.t('Made by OpenWebUI Community')}
	</div>

	<a
		class=" flex space-x-4 cursor-pointer w-full mb-2 px-3 py-2"
		href="https://openwebui.com/#open-webui-community"
		target="_blank"
	>
		<div class=" self-center w-10 flex-shrink-0">
			<div
				class="w-full h-10 flex justify-center rounded-full bg-transparent dark:bg-gray-700 border border-dashed border-gray-200"
			>
				<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="w-6">
					<path
						fill-rule="evenodd"
						d="M12 3.75a.75.75 0 01.75.75v6.75h6.75a.75.75 0 010 1.5h-6.75v6.75a.75.75 0 01-1.5 0v-6.75H4.5a.75.75 0 010-1.5h6.75V4.5a.75.75 0 01.75-.75z"
						clip-rule="evenodd"
					/>
				</svg>
			</div>
		</div>

		<div class=" self-center">
			<div class=" font-semibold line-clamp-1">{$i18n.t('Discover a form')}</div>
			<div class=" text-sm line-clamp-1">
				{$i18n.t('Discover, download, and explore custom forms')}
			</div>
		</div>
	</a>
</div>

<DeleteConfirmDialog
	bind:show={showDeleteConfirm}
	title={$i18n.t('Delete form?')}
	on:confirm={() => {
		deleteHandler(deleteForm);
	}}
>
	<div class=" text-sm text-gray-500">
		{$i18n.t('This will delete')} <span class="  font-semibold">{deleteForm.command}</span>.
	</div>
</DeleteConfirmDialog>
