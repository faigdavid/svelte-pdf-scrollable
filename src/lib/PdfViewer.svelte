<script lang="ts">
	import { onMount } from 'svelte';
	import * as pdfjs from 'pdfjs-dist';
	import pdfjsWorker from 'pdfjs-dist/build/pdf.worker.min.js?url';
	import FileSaver from 'file-saver';
	import downloadsvg from './images/toolbarDownload.svg?url';
	import printsvg from './images/toolbarPrint.svg?url';
	import zoominsvg from './images/toolbarZoomIn.svg?url';
	import zoomoutsvg from './images/toolbarZoomOut.svg?url';
	import spreadsvg from './images/toolbarPageView.svg?url';
	import gapsvg from './images/toolbarPageGap.svg?url';
	import './pdfviewer.css';

	pdfjs.GlobalWorkerOptions.workerSrc = pdfjsWorker;

	export let url: string | URL; //url of pdf.
	const INTERNAL_URL = url.toString();

	let classname = ''; //allows component to recieve classes
	export { classname as class };

	let styles = ''; //allows component to recieve classes
	export { styles as style };

	export let scale = 1;
	const MIN_SCALE = 0.5;
	const MAX_SCALE = 2.3;

	enum SpreadModes { //init display modes.
		'NONE',
		'ODD',
		'EVEN',
	}
	export let display_mode = '';
	let _spread_mode = SpreadModes.NONE;
	if (display_mode in SpreadModes) {
		_spread_mode = SpreadModes[display_mode as 'NONE' | 'ODD' | 'EVEN'];
	}

	//internal variables.
	let component_container: HTMLDivElement;
	let container: HTMLDivElement;
	let password = '';
	let password_error = false;
	let password_message = '';
	let _prev_gap_top = '8px';
	let _prev_gap_bottom = '8px';

	//Init button handlers (some require hydration on mount)
	let onPasswordSubmit = () => {};
	let onZoomIn = () => {};
	let onZoomOut = () => {};
	let onPageDisplay = () => {};

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

	const downloadPdf = (file_url: string) => {
		const filename = file_url.substring(file_url.lastIndexOf('/') + 1);
		FileSaver.saveAs(file_url, filename);
	};

	const onPageGap = () => {
		const pages = component_container.getElementsByClassName('page');
		if (pages.length === 0) {
			return;
		}
		const current_styles = getComputedStyle(pages[0] as HTMLDivElement);
		const current_gap_bottom = current_styles.marginBottom;
		const current_gap_top = current_styles.marginTop;
		for (const page of pages) {
			(page as HTMLDivElement).style.marginBottom = _prev_gap_bottom;
			(page as HTMLDivElement).style.marginTop = _prev_gap_top;
		}
		_prev_gap_bottom = current_gap_bottom;
		_prev_gap_top = current_gap_top;
	};

	onMount(() => {
		if (['static', 'initial'].includes(getComputedStyle(component_container).position)) {
			console.warn('PdfViewer sizing only works when it is positioned (not static).');
		}
		const init_promise = import('pdfjs-dist/web/pdf_viewer.js').then((pdfjs_viewer) => {
			const event_bus = new pdfjs_viewer.EventBus();

			// (Optionally) enable hyperlinks within PDF files.
			const pdf_link_service = new pdfjs_viewer.PDFLinkService({
				eventBus: event_bus,
			});

			// (Optionally) enable find controller.
			const pdf_find_controller = new pdfjs_viewer.PDFFindController({
				eventBus: event_bus,
				linkService: pdf_link_service,
			});
			const pdf_viewer = new pdfjs_viewer.PDFViewer({
				container,
				eventBus: event_bus,
				linkService: pdf_link_service,
				findController: pdf_find_controller,
				l10n: pdfjs_viewer.NullL10n,
			});
			pdf_link_service.setViewer(pdf_viewer);
			return { pdf_viewer, pdf_link_service };
		});

		const renderDocument = async () => {
			const { pdf_viewer, pdf_link_service } = await init_promise;
			// Loading document.
			const loading_task = pdfjs.getDocument({
				url,
				password,
			});
			loading_task.promise
				.then((pdf_document) => {
					pdf_viewer.setDocument(pdf_document);
					pdf_link_service.setDocument(pdf_document, null);
					pdf_viewer.currentScale = scale;
					pdf_viewer.spreadMode = _spread_mode;
				})
				.catch(function (error) {
					password_error = true;
					password_message = error.message;
				});

			onZoomIn = () => {
				if (scale <= MAX_SCALE) {
					scale = scale + 0.1;
					pdf_viewer.currentScale = scale;
				}
			};
			onZoomOut = () => {
				if (scale >= MIN_SCALE) {
					scale = scale - 0.1;
					pdf_viewer.currentScale = scale;
				}
			};
			onPageDisplay = () => {
				_spread_mode = (_spread_mode + 1) % 3;
				pdf_viewer.spreadMode = _spread_mode;
			};
			return pdf_viewer;
		};
		const render = renderDocument();

		onPasswordSubmit = () => {
			renderDocument();
		};

		return () => {
			render.then((pdf_viewer) => {
				pdf_viewer.cleanup();
			});
		};
	});
</script>

<div class={classname} style={styles} bind:this={component_container}>
	<div id="viewer-parent" class="w-full h-full">
		{#if password_error === true}
			<div class="spdfinner">
				<p>This document requires a password to open:</p>
				<p>{password_message}</p>
				<div>
					<input type="password" bind:value={password} />
					<button on:click={onPasswordSubmit} class="password-button"> Submit </button>
				</div>
			</div>
		{:else}
			<div class="spdfbanner">
				<span class="toolbarbutton" on:click={onZoomIn}>
					<img title="Zoom In" src={zoominsvg} alt="zoom in" class="spdfbutton" />
				</span>
				<span class="toolbarbutton" on:click={onZoomOut}>
					<img title="Zoom Out" src={zoomoutsvg} alt="zoom out" class="spdfbutton" />
				</span>
				<span class="toolbarbutton" on:click={onPageGap}>
					<img title="Toggle Page Display" src={gapsvg} alt="toggle page gap" class="spdfbutton" />
				</span>
				<span class="toolbarbutton" on:click={onPageDisplay}>
					<img
						title="Toggle Page Display"
						src={spreadsvg}
						alt="toggle page display"
						class="spdfbutton"
					/>
				</span>
				<span class="toolbarbutton" on:click={() => printPdf(INTERNAL_URL)}>
					<img title="Print" src={printsvg} alt="print" class="spdfbutton" />
				</span>
				<span class="toolbarbutton" on:click={() => downloadPdf(INTERNAL_URL)}>
					<img title="Download" src={downloadsvg} alt="download" class="spdfbutton" />
				</span>
			</div>
			<div class="spdfinner">
				<div id="viewerContainer" style="background-color: rgb(55 65 81);" bind:this={container}>
					<div id="viewer" class="pdfViewer" />
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	.spdfbanner {
		position: absolute;
		z-index: 10;
		top: 0px;
		height: 2.75rem;
		width: 100%;
		display: flex;
		justify-content: center;
		gap: 0.5rem;
		align-items: center;
		background-color: rgb(41 37 36);
		box-shadow: 1rem;
	}
	.spdfbutton {
		width: 25px;
	}
	.spdfinner {
		position: absolute;
		top: 0px;
		bottom: 0px;
		width: 100%;
	}
	.toolbarbutton:hover {
		background-color: rgb(87 83 78);
	}
	.toolbarbutton {
		border-radius: 2px;
		padding: 4px;
	}
</style>
