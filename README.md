<div align="center">
<img width="1200" height="475" alt="GHBanner" src="https://github.com/user-attachments/assets/0aa67016-6eaf-458a-adb2-6e31a0763ed6" />
</div>

# Run and deploy your AI Studio app

This contains everything you need to run your app locally.

View your app in AI Studio: https://ai.studio/apps/1bc9ad02-3e7b-4c7c-a0b5-1b2aede460bb

## Run Locally

**Prerequisites:**  Node.js


1. Install dependencies:
   `npm install`
2. Set the `GEMINI_API_KEY` in [.env.local](.env.local) to your Gemini API key
3. Run the app:
   `npm run dev`





# RFecho PDF Intelligent Extraction and Generation Tool
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An automated PDF processing tool tailored for the electronic component industry. It can batch extract product information (tables, images, text) from PDFs and generate standardized English product datasheets based on customizable templates.

## Core Features
### 1. Intelligent Information Extraction
- **Text Extraction**: Automatically identify and extract product names, features, descriptions, technical specification tables, and environmental parameter tables.
- **Image Extraction**: Precisely crop product appearance images (main), engineering structure diagrams (outline), and performance curves (Curves).
- **Multilingual Processing**: Automatically translate Chinese content into professional English technical text.
- **Special Symbol Repair**: Automatically correct display issues of technical symbols such as µA/℃/Ω.

### 2. Customizable PDF Generation
- **Standardized Headers and Footers**: Support custom Logo/background images and fixed footer copyright information.
- **Flexible Layout Rules**:
  - Page 1: Product name + main image + two-column layout for features/description.
  - From Page 2: Technical tables + engineering drawings + performance curves.
  - Adaptive image scaling to maintain original proportions without page overflow.
- **Manual Image Override**: Support uploading custom images to replace automatically extracted ones.
- **Multi-Image Processing**: Support adaptive layout for multiple main/outline images.

### 3. Batch Processing Capability
- **Folder-Level Batch Upload**: Support batch processing of PDFs with nested folder structures.
- **Automatic Image Matching**: Automatically match image materials for corresponding products by folder name.
- **Memory Optimization**: Support processing large file scenarios with 20+ images per product.

### 4. Product Naming Rules
- Automatic Renaming: Replace product names starting with H with O, and add O as a prefix to other product names (customizable).
- Generated PDFs automatically inherit the original product names.

## Quick Start
### Environment Requirements
- Node.js ≥ 16.x
- npm/yarn/pnpm
- Modern browsers (Chrome/Firefox/Edge)
- Google Gemini API Key (Gemini 3 Flash Preview permission required)

### Installation Steps
1. Clone the repository
```bash
git clone https://github.com/your-username/rfecho-pdf-generator.git
cd rfecho-pdf-generator
```

2. Install dependencies
```bash
npm install
# or yarn install
# or pnpm install
```

3. Configure API Key
Replace the following content in the `index.tsx` file:
```typescript
const GEMINI_API_KEY = "YOUR_GEMINI_API_KEY"; // Replace with your Gemini API Key
```

4. Start the development server
```bash
npm run dev
# or yarn dev
# or pnpm dev
```

5. Access the application
Open your browser and visit `http://localhost:5173` (default port)

### Usage Guide
#### Single File Processing
1. Upload header and footer materials (logo.png/top.png/Botom.png) - only need to upload once.
2. Upload the PDF file to be processed.
3. (Optional) Upload custom images (main/outline/curves).
4. Click the "Process PDF" button and download after generation is complete.

#### Batch Processing
1. Upload header and footer materials (logo.png/top.png/Botom.png).
2. Upload the folder containing all PDFs (supports nested structures).
3. Upload the image folder (subfolder names must match PDF filenames).
4. Click the "Batch Process" button, and the program will automatically process all files and allow batch download.

## Code Structure and Custom Development
### Core File Description
| File | Function Description |
|------|---------------------|
| `index.tsx` | Main program entry, containing core logic for PDF processing, AI calls, and PDF generation. |
| `App.tsx` | UI component responsible for file upload, status display, and interaction logic. |
| `Layout.tsx` | PDF layout template defining headers/footers and page structure. |
| `ScannerMode.tsx` | UI for single file processing mode. |
| `StoryMode.tsx` | UI for batch processing mode. |
| `main.txt` | Python reference code (historical implementation, for reference only). |

### Custom Modification Guide
#### 1. Modify PDF Template Styles
**Goal**: Adjust headers/footers, fonts, colors, and layout.
**Modification Location**: `generatePDF` function in `index.tsx`
```typescript
// Adjust header Logo position (example: modify Logo position and size)
doc.addImage(logoData, 'PNG', 10, 10, 50, 20); // x, y, width, height
// Original logic: maintain original Logo proportions without scaling
doc.addImage(logoData, 'PNG', 5, 5, logoWidth, logoHeight); // Shift to top-left, use original size

// Modify footer text
const footerLines = [
  'RFecho is trademark of Ocean Microwave',
  'All rights of respective trademark owners reserved.',
  '©RFecho2025'
];
// Modify footer text/position/font:
doc.setFontSize(8);
doc.setTextColor(50, 50, 50);
footerLines.forEach((line, i) => {
  doc.text(line, doc.internal.pageSize.width - 10, doc.internal.pageSize.height - 10 - (i * 5), { align: 'right' });
});

// Font/color modification
doc.setFont('Helvetica', 'bold'); // Font/style
doc.setFontSize(16); // Font size
doc.setTextColor(33, 33, 33); // Color (RGB)
```

#### 2. Adjust Information Extraction Rules
**Goal**: Modify content/rules for AI extraction.
**Modification Location**: Gemini Prompt in the `processPDF` function in `index.tsx`
```typescript
const prompt = `
  You are a professional electronic component technical document analyst. Please complete the following tasks:
  1. Extract product name: ${productNameExtractionRule}
  2. Extract feature list: ${featuresExtractionRule}
  3. Extract description text: ${descriptionExtractionRule}
  // Custom extraction rule example:
  4. Ignore the "Pin Configuration" table (add/remove ignore rules as needed)
  5. Extract the "Absolute Maximum Ratings" table (add new table types as needed)
`;
```

#### 3. Modify Image Processing Logic
**Goal**: Adjust image scaling/layout rules.
**Modification Location**: `addImageToPDF` function in `index.tsx`
```typescript
// Outline image scaling rule (example: adaptive to page width)
function addOutlineImages(doc, outlineImages) {
  outlineImages.forEach(img => {
    doc.addPage();
    const pageWidth = doc.internal.pageSize.width - 40; // 20mm margins on left and right
    const scale = pageWidth / img.width;
    const newHeight = img.height * scale;
    // Center display without page overflow
    doc.addImage(img.data, 'PNG', 20, 40, pageWidth, newHeight);
  });
}

// Curve image scaling rule for last image (example: scale up when aspect ratio > 1.8)
function addCurveImages(doc, curveImages) {
  curveImages.forEach((img, index) => {
    let width = img.width * 0.8;
    let height = img.height * 0.8;
    // Scale up last image when aspect ratio > 1.8
    if (index === curveImages.length - 1 && img.width / img.height > 1.8) {
      width = Math.min(width * 2, doc.internal.pageSize.width - 40);
      height = img.height * (width / img.width);
    }
    doc.addImage(img.data, 'PNG', 20, 40, width, height);
  });
}
```

#### 4. Modify Batch Processing Logic
**Goal**: Adjust folder structure/file matching rules.
**Modification Location**: `handleBulkUpload` function in `index.tsx`
```typescript
// Custom file matching rule (example: support different file suffixes)
function matchPdfWithImages(pdfFiles, imageFolders) {
  return pdfFiles.map(pdf => {
    const pdfName = pdf.name.replace(/\.pdf$/i, '');
    // Custom matching rule: support underscore/space replacement
    const matchedFolder = imageFolders.find(folder => 
      folder.name.replace(/_/g, ' ').toLowerCase() === pdfName.toLowerCase()
    );
    return { pdf, images: matchedFolder?.files || [] };
  });
}
```

#### 5. Modify Product Naming Rules
**Goal**: Adjust product renaming logic.
**Modification Location**: `transformProductName` function in `index.tsx`
```typescript
// Custom naming rule (example: remove prefix/suffix)
function transformProductName(originalName) {
  // Original rule: H→O, add O to others
  // Custom rule example: uppercase all, remove numeric suffix
  let newName = originalName.toUpperCase().replace(/\d+$/, '');
  // Add custom logic as needed
  return newName;
}
```

#### 6. Resolve Description Layout Issues
**Goal**: Fix letter spacing/font inconsistency issues.
**Modification Location**: `cleanDescription` and `drawDescription` functions in `index.tsx`
```typescript
// Text cleaning: remove Markdown/excess spaces
function cleanDescription(text) {
  return text
    .replace(/\*\*|#|Description:/gi, '') // Remove Markdown tags
    .replace(/\s+/g, ' ') // Merge excess spaces
    .trim();
}

// Draw description text: force uniform font/alignment
function drawDescription(doc, text, x, y, width) {
  doc.setFont('Helvetica', 'normal'); // Uniform font
  doc.setFontSize(10); // Uniform font size
  doc.setTextColor(33, 33, 33); // Uniform color
  const lines = doc.splitTextToSize(text, width); // Auto line break
  doc.text(lines, x, y, { 
    align: 'left', // Force left alignment
    characterSpacing: 0, // Reset letter spacing
    lineHeightFactor: 1.4 // Uniform line height
  });
}
```

#### 7. Handle Large Number of Images (20+)
**Goal**: Optimize memory usage and support processing large number of images.
**Modification Location**: `processImages` function in `index.tsx`
```typescript
// Asynchronous batch image processing to avoid memory overflow
async function processImages(imageFiles) {
  const processedImages = [];
  // Process one by one instead of all at once
  for (const file of imageFiles) {
    const imgData = await convertFileToBase64(file);
    const img = new Image();
    img.src = imgData;
    await new Promise(resolve => img.onload = resolve);
    processedImages.push({
      data: imgData,
      width: img.width,
      height: img.height,
      name: file.name
    });
    // Release memory
    URL.revokeObjectURL(img.src);
  }
  return processedImages;
}
```

## Troubleshooting
### 1. API Errors (404/500)
- Verify that the Gemini API Key is valid and has correct permissions.
- Check that the model name is correct (recommended: `gemini-3-flash-preview`).
- Reduce PDF rendering resolution (from 2.5x to 1.5x).

### 2. Incomplete Table Extraction
- Optimize the Gemini Prompt to explicitly request extraction of all table types.
- Increase PDF rendering resolution to improve OCR accuracy.
- Check if the PDF is a scanned document (additional OCR processing required for scanned documents).

### 3. Image Overflow/Scaling Abnormalities
- Adjust the scaling factor in `addImageToPDF`.
- Add page boundary check logic.
- Ensure correct calculation of original image proportions.

### 4. Memory Overflow in Batch Processing
- Enable asynchronous image processing (see above).
- Limit the number of files processed at once.
- Increase browser memory limit (Chrome: `--max-old-space-size=4096`).

## License
This project is open source under the MIT License. See the [LICENSE](LICENSE) file for details.

## Disclaimer
- This tool is only for legal technical document processing and prohibited for infringing use.
- Ensure you have the right to process PDF files before use.
- The author is not responsible for any data loss/format errors caused by the use of this tool.

## Contribution Guide
1. Fork the repository.
2. Create a feature branch (`git checkout -b feature/amazing-feature`).
3. Commit your changes (`git commit -m 'Add some amazing feature'`).
4. Push to the branch (`git push origin feature/amazing-feature`).
5. Open a Pull Request.

## Contact
For questions/suggestions, please contact:
- GitHub Issues: [Submit an issue](https://github.com/your-username/rfecho-pdf-generator/issues)
- Email: your-email@example.com
