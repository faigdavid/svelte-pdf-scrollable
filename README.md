# svelte-pdf-scrollable

A sveltekit component that can render PDF files using pdfjs. Since the PDFjs official documentation doesn't really bridge the gap from the examples to the demo viewer, I decided to make something in-between for hobbiests like me to work with.

I have not tried to get this working on svelte only! I relied heavily on vite's import capabilities.

Some other svelte pdf components are out there but they are all single-page viewers, so I decided to build a scrollable viewer. I have not yet included a way to toggle between scroll-mode and page-mode.

Comes with several buttons:

- Zoom In / Zoom Out
- Toggle Page Gap
- Toggle Spread Mode
- Print
- Download

Feel free to take the code apart and use it however you like!

## Install

If you are making a new svelte-kit project, refer to https://kit.svelte.dev/docs/introduction#getting-started

```
npm install svelte-pdf-scrollable
```

From there you can simply import and use the viewer.

```
<script>
    import PdfViewer from 'svelte-pdf-scrollable';
</script>

<PdfViewer
	url="/my_pdf_name.pdf"
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
    scale={0.8}
/>
```

Make sure your `<PdfViewer>` styling is not `position: static`. Otherwise it will not size itself properly within its container.

## Demo

To see a simple demo of the pdf component, build and run from the repository:

```
git clone https://github.com/faigdavid/svelte-pdf-scrollable

cd svelte-pdf-scrollable

npm install

npm run dev
```

From there you should be able to see the demo at `http://localhost:5173/`
