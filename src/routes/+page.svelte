<script lang="ts">
	import PdfViewer from '$lib/PdfViewer.svelte';
</script>

<h1 style="width:max-content; margin: auto; font-size: 20px;">
	Check out my PDF on the logical definability of Tree Languages!
</h1>

<PdfViewer
	url="/D_Faig_report_vf.pdf"
	style="
        position: absolute;
        top: 5rem;
        left: 50%;
        transform: translate(-50%);
        width: 900px;
        height: 800px;
        outline-style: solid;
        outline-width: 1px;
        outline-offset: 0px;
    "
/>
