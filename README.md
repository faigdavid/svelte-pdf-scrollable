# svelte-pdf-scrollable

A svelte component that can render PDF files using pdfjs. Since the PDFjs official documentation doesn't really bridge the gap from the examples to the demo viewer, I decided to make something in-between for hobbiests like me to work with.

Some other svelte pdf components are out there but they are all single-page viewers, so I decided to build a scrollable viewer. I have not yet included a way to toggle between scroll-mode and page-mode.

Comes with several buttons:

- Zoom In / Zoom Out
- Toggle Page Gap
- Toggle Spread Mode
- Print
- Download

Feel free to take the code apart and use it however you like!

## Demo

To see a simple demo of the pdf component, build and run from the repository:

```
git clone https://github.com/faigdavid/svelte-pdf-scrollable

cd svelte-pdf-scrollable

npm install

npm run dev
```

From there you should be able to see it at `http://localhost:5173/`
