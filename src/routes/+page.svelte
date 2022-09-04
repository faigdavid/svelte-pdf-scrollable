<script lang="ts">
	import PdfViewer from '$lib/PdfViewer.svelte';
	import '../app.css';
</script>

<h1 class="w-fit m-auto text-2xl font-bold">
	Check out my PDF on the logical definability of Tree Languages!
</h1>
<PdfViewer
	url="/D_Faig_report_vf.pdf"
	class="absolute top-20 left-[15%] right-[15%] bottom-0 outline outline-1 outline-offset-0"
/>
