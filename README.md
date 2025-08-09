# Google Drive Protected PDF Downloader

This project provides a method to download view-only protected PDF files from Google Drive using a browser script. The script utilizes the `jsPDF` library to convert images into a downloadable PDF format.

## Features

- Downloads view-only protected PDFs from Google Drive.
- Converts images from the page into a single PDF file.
- Utilizes the Trusted Types API for secure script loading.

## Requirements

- A modern web browser that supports the Trusted Types API and JavaScript.
- Access to view the PDF files on Google Drive.

## Usage

1. **Open the PDF in Google Drive:** Navigate to the PDF file you want to download.
2. **Run the Script:**
   - Open your browser's developer tools (usually F12 or right-click > Inspect).
   - Go to the Console tab.
   - Copy and paste the entire script below into the console and press Enter.

```javascript
// === Drive/Viewer -> PDF (high-res, auto-orientation, fit-to-page) ===
(() => {
  const FIXED_PAGE_FORMAT = null; // e.g., [1920, 1080] for fixed 16:9; otherwise null
  const MARGIN_PX = 0;

  function makePdf() {
    const jsPDF = (window.jspdf && window.jspdf.jsPDF) || window.jsPDF;
    if (!jsPDF) throw new Error('jsPDF not available after load.');

    // 1) Collect blob-backed images that represent pages
    const imgs = Array.from(document.images).filter(img => /^blob:/.test(img.src));
    if (!imgs.length) throw new Error('No blob-backed images found on the page.');

    // Keep visual order
    imgs.sort((a, b) => {
      const ra = a.getBoundingClientRect();
      const rb = b.getBoundingClientRect();
      return ra.top - rb.top || ra.left - rb.left;
    });

    // 2) Initial page format/orientation
    const first = imgs[0];
    const fw = first.naturalWidth || first.width;
    const fh = first.naturalHeight || first.height;
    const firstOrientation = fw >= fh ? 'landscape' : 'portrait';
    const baseFormat = Array.isArray(FIXED_PAGE_FORMAT) ? FIXED_PAGE_FORMAT : [fw, fh];

    const pdf = new jsPDF({ orientation: firstOrientation, unit: 'px', format: baseFormat });

    // 3) Canvas for high-quality capture
    const canvas = document.createElement('canvas');
    const ctx = canvas.getContext('2d', { alpha: false });
    ctx.imageSmoothingEnabled = false;

    imgs.forEach((img, i) => {
      const w = img.naturalWidth || img.width;
      const h = img.naturalHeight || img.height;

      if (i > 0) {
        const pageFormat = Array.isArray(FIXED_PAGE_FORMAT) ? FIXED_PAGE_FORMAT : [w, h];
        const orient = (pageFormat[0] >= pageFormat[1]) ? 'landscape' : 'portrait';
        pdf.addPage(pageFormat, orient);
      }

      const pageW = pdf.internal.pageSize.getWidth();
      const pageH = pdf.internal.pageSize.getHeight();

      const maxW = pageW - MARGIN_PX * 2;
      const maxH = pageH - MARGIN_PX * 2;

      const scale = Math.min(maxW / w, maxH / h);
      const drawW = Math.floor(w * scale);
      const drawH = Math.floor(h * scale);

      const x = (pageW - drawW) / 2;
      const y = (pageH - drawH) / 2;

      canvas.width = w;
      canvas.height = h;
      ctx.clearRect(0, 0, w, h);
      ctx.drawImage(img, 0, 0, w, h);

      const imgData = canvas.toDataURL('image/jpeg', 1.0);
      pdf.addImage(imgData, 'JPEG', x, y, drawW, drawH);
    });

    pdf.save('download.pdf');
  }

  // If jsPDF is already present, use it; otherwise load once.
  if (window.jspdf?.jsPDF || window.jsPDF) {
    try { makePdf(); } catch (e) { console.error(e); }
    return;
  }

  // Trusted Types-safe loader
  const ttPolicy = window.trustedTypes?.createPolicy?.('myTrustedTypesPolicy', {
    createScriptURL: (input) => input
  });

  const url = 'https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js';
  const safeUrl = ttPolicy ? ttPolicy.createScriptURL(url) : url;

  const tag = document.createElement('script');
  tag.onload = () => {
    try { makePdf(); } catch (e) { console.error('Error generating PDF:', e); }
  };
  tag.onerror = () => console.error('Failed to load jsPDF from CDN.');
  tag.src = safeUrl;
  document.body.appendChild(tag);
})();

```

3. **Download the PDF:** The script will generate a PDF file containing the images from the Google Drive PDF and prompt you to download it.

## Disclaimer

- Use this script responsibly and only for personal use or with permission from the content owner.
- The authors of this project do not take any responsibility for any misuse of the script.

## Contributing

If you have suggestions or improvements, feel free to create a pull request or open an issue.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
