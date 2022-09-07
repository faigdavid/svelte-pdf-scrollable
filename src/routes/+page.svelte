<script lang="ts">
	import PdfViewer from '$lib/PdfViewer.svelte';
	import '../app.css';
</script>

<h1 class="w-fit m-auto text-2xl font-bold">
	Check out my PDF on the logical definability of Tree Languages!
</h1>
<PdfViewer
	url="/D_Faig_report_vf.pdf"
	class="absolute top-20 left-1/2 -translate-x-1/2 w-[900px] h-[800px] outline outline-1 outline-offset-0"
/>
