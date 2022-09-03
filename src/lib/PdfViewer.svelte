<script lang="ts">
	import { onMount } from 'svelte';
	import * as pdfjs from 'pdfjs-dist';
	import pdfjsWorker from 'pdfjs-dist/build/pdf.worker.min.js?url';
	import FileSaver from 'file-saver';
	import { Tooltip } from './utils';
	import downloadsvg from './images/toolbarDownload.svg?url';
	import printsvg from './images/toolbarPrint.svg?url';
	import zoominsvg from './images/toolbarZoomIn.svg?url';
	import zoomoutsvg from './images/toolbarZoomOut.svg?url';
	import './pdfviewer.css';

	export let url: string | URL;
	export let scale = 1;
	let classname = '';
	export { classname as class };

	const internalURL = url.toString();

	pdfjs.GlobalWorkerOptions.workerSrc = pdfjsWorker;

	let container: HTMLDivElement;
	let password = '';
	let passwordError = false;
	let passwordMessage = '';
	const minScale = 1.0;
	const maxScale = 2.3;

	let onPasswordSubmit = () => {};
	let onZoomIn = () => {};
	let onZoomOut = () => {};

	const printPdf = (url: string) => {
		const iframe = document.createElement('iframe');
		document.body.appendChild(iframe);

		iframe.style.display = 'none';
		iframe.onload = function () {
			setTimeout(function () {
				iframe.focus();
				iframe.contentWindow?.print();
			}, 1);
		};

		iframe.src = url;
	};

	const downloadPdf = (fileURL: string) => {
		let fileName = fileURL.substring(fileURL.lastIndexOf('/') + 1);
		FileSaver.saveAs(fileURL, fileName);
	};

	onMount(() => {
		const initPromise = import('pdfjs-dist/web/pdf_viewer.js').then((pdfjsViewer) => {
			const eventBus = new pdfjsViewer.EventBus();

			// (Optionally) enable hyperlinks within PDF files.
			const pdfLinkService = new pdfjsViewer.PDFLinkService({
				eventBus,
			});

			// (Optionally) enable find controller.
			const pdfFindController = new pdfjsViewer.PDFFindController({
				eventBus,
				linkService: pdfLinkService,
			});
			console.log(container);
			const pdfViewer = new pdfjsViewer.PDFViewer({
				container,
				eventBus,
				linkService: pdfLinkService,
				findController: pdfFindController,
				l10n: pdfjsViewer.NullL10n,
			});
			pdfViewer.currentScale = scale;
			pdfLinkService.setViewer(pdfViewer);
			return { pdfViewer, pdfLinkService };
		});

		const renderDocument = async () => {
			const { pdfViewer, pdfLinkService } = await initPromise;
			// Loading document.
			const loadingTask = pdfjs.getDocument({
				url,
				password,
			});
			loadingTask.promise
				.then((pdfDocument) => {
					pdfViewer.setDocument(pdfDocument);
					pdfLinkService.setDocument(pdfDocument, null);
				})
				.catch(function (error) {
					passwordError = true;
					passwordMessage = error.message;
				});

			onZoomIn = () => {
				if (scale <= maxScale) {
					scale = scale + 0.1;
					pdfViewer.currentScale = scale;
				}
			};
			onZoomOut = () => {
				if (scale >= minScale) {
					scale = scale - 0.1;
					pdfViewer.currentScale = scale;
				}
			};

			return pdfViewer;
		};
		const render = renderDocument();

		onPasswordSubmit = () => {
			renderDocument();
		};

		return () => {
			render.then((pdfViewer) => {
				pdfViewer.cleanup();
			});
		};
	});
</script>

<div id="viewer-parent" lang="en-US" class={classname}>
	<body>
		{#if passwordError === true}
			<div class="absolute top-0 bottom-0 w-full">
				<p>This document requires a password to open:</p>
				<p class="">{passwordMessage}</p>
				<div class="">
					<input type="password" class="" bind:value={password} />
					<button on:click={onPasswordSubmit} class="password-button"> Submit </button>
				</div>
			</div>
		{:else}
			<div
				class="absolute z-10 top-0 h-11 w-full flex justify-center gap-4 items-center bg-stone-800 drop-shadow-sm"
			>
				<Tooltip name="Zoom In">
					<span on:click={onZoomIn}>
						<img src={zoominsvg} alt="zoom in button" class="w-7 p-1 hover:bg-stone-600" />
					</span>
				</Tooltip>
				<Tooltip name="Zoom Out">
					<span on:click={onZoomOut}>
						<img src={zoomoutsvg} alt="zoom out button" class="w-7 p-1 hover:bg-stone-600" />
					</span>
				</Tooltip>
				<Tooltip name="Print">
					<span on:click={() => printPdf(internalURL)}>
						<img src={printsvg} alt="print button" class="w-7 p-1 hover:bg-stone-600" />
					</span>
				</Tooltip>
				<Tooltip name="Download">
					<span on:click={() => downloadPdf(internalURL)}>
						<img src={downloadsvg} alt="download button" class="w-7 p-1 hover:bg-stone-600" />
					</span>
				</Tooltip>
			</div>
			<div class="absolute top-0 bottom-0 w-full">
				<div id="viewerContainer" class="bg-gray-700" bind:this={container}>
					<div id="viewer" class="pdfViewer" />
				</div>
			</div>
		{/if}
	</body>
</div>
