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
const trustedTypesPolicy = window.trustedTypes.createPolicy('myTrustedTypesPolicy', {
    createScriptURL: (input) => input
});

let jspdfSrc = trustedTypesPolicy.createScriptURL('https://cdnjs.cloudflare.com/ajax/libs/jspdf/1.5.3/jspdf.debug.js');

let jspdf = document.createElement("script");

jspdf.onload = function () {
    try {
        let pdf = new jsPDF();
        let elements = document.getElementsByTagName("img");
        for (let img of elements) {
            if (!/^blob:/.test(img.src)) continue;

            let canvasElement = document.createElement('canvas');
            let con = canvasElement.getContext("2d");
            canvasElement.width = img.width;
            canvasElement.height = img.height;
            con.drawImage(img, 0, 0, img.width, img.height);
            let imgData = canvasElement.toDataURL("image/jpeg", 1.0);
            pdf.addImage(imgData, 'JPEG', 0, 0);
            pdf.addPage();
        }
        pdf.save("download.pdf");
    } catch (error) {
        console.error("Error generating PDF:", error);
    }
};

jspdf.src = jspdfSrc;
document.body.appendChild(jspdf);
```

3. **Download the PDF:** The script will generate a PDF file containing the images from the Google Drive PDF and prompt you to download it.

## Disclaimer

- Use this script responsibly and only for personal use or with permission from the content owner.
- The authors of this project do not take any responsibility for any misuse of the script.

## Contributing

If you have suggestions or improvements, feel free to create a pull request or open an issue.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.