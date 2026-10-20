<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>PI Message Generator</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.8.0/mammoth.browser.min.js"></script>

<style>
    * {
        box-sizing: border-box;
    }

    body {
        margin: 0;
        font-family: Arial, sans-serif;
        background: #f5f7fa;
        color: #222;
    }

    .container {
        max-width: 1100px;
        margin: 40px auto;
        padding: 0 20px 50px;
    }

    h1 {
        margin-bottom: 8px;
    }

    .subtitle {
        color: #666;
        margin-bottom: 30px;
    }

    .card {
        background: white;
        border-radius: 12px;
        padding: 24px;
        margin-bottom: 20px;
        box-shadow: 0 2px 10px rgba(0,0,0,0.06);
    }

    h2 {
        margin-top: 0;
        font-size: 20px;
    }

    input[type="file"] {
        width: 100%;
        padding: 12px;
        border: 1px solid #ddd;
        border-radius: 8px;
        background: #fafafa;
    }

    button {
        border: none;
        border-radius: 8px;
        padding: 12px 18px;
        cursor: pointer;
        font-size: 15px;
        margin-top: 12px;
        margin-right: 8px;
    }

    .primary {
        background: #2563eb;
        color: white;
    }

    .secondary {
        background: #e5e7eb;
        color: #222;
    }

    button:hover {
        opacity: 0.9;
    }

    .status {
        margin-top: 12px;
        color: #555;
        min-height: 20px;
    }

    .grid {
        display: grid;
        grid-template-columns: 1fr 1fr;
        gap: 15px;
    }

    .full {
        grid-column: 1 / -1;
    }

    label {
        display: block;
        font-weight: bold;
        margin-bottom: 6px;
        font-size: 14px;
    }

    input,
    textarea {
        width: 100%;
        border: 1px solid #d1d5db;
        border-radius: 7px;
        padding: 10px;
        font-family: inherit;
        font-size: 14px;
    }

    textarea {
        min-height: 80px;
        resize: vertical;
    }

    #products {
        min-height: 130px;
    }

    #message {
        min-height: 420px;
        font-family: monospace;
        white-space: pre-wrap;
    }

    .hidden {
        display: none;
    }

    .notice {
        background: #fff7ed;
        border: 1px solid #fed7aa;
        padding: 12px;
        border-radius: 8px;
        color: #9a3412;
        margin-top: 15px;
        font-size: 14px;
    }

    @media (max-width: 700px) {
        .grid {
            grid-template-columns: 1fr;
        }

        .full {
            grid-column: auto;
        }
    }
</style>
</head>

<body>

<div class="container">

    <h1>PI Message Generator</h1>
    <div class="subtitle">
        Upload a Proforma Invoice and generate a ready-to-copy supply chain message.
    </div>

    <div class="card">
        <h2>1. Upload Proforma Invoice</h2>

        <input
            type="file"
            id="fileInput"
            accept=".pdf,.docx"
        >

        <button class="primary" onclick="extractInvoice()">
            Extract Invoice
        </button>

        <div id="status" class="status"></div>

        <div class="notice">
            Stock Location is intentionally left blank when it is not stated in the PI.
            You can enter it manually after extraction.
        </div>
    </div>


    <div class="card">
        <h2>2. Review / Edit Information</h2>

        <div class="grid">

            <div>
                <label>Bill To</label>
                <input id="billTo">
            </div>

            <div>
                <label>Bill To Attn</label>
                <input id="billAttn">
            </div>

            <div class="full">
                <label>Bill To Address</label>
                <textarea id="billAddress"></textarea>
            </div>

            <div>
                <label>Bill To Email</label>
                <input id="billEmail">
            </div>

            <div>
                <label>Bill To Telephone</label>
                <input id="billPhone">
            </div>


            <div>
                <label>Ship To</label>
                <input id="shipTo">
            </div>

            <div>
                <label>Ship To Attn</label>
                <input id="shipAttn">
            </div>

            <div class="full">
                <label>Ship To Address</label>
                <textarea id="shipAddress"></textarea>
            </div>

            <div>
                <label>Ship To Email</label>
                <input id="shipEmail">
            </div>

            <div>
                <label>Ship To Telephone</label>
                <input id="shipPhone">
            </div>


            <div class="full">
                <label>Products</label>
                <textarea id="products"></textarea>
            </div>

            <div class="full">
                <label>Terms</label>
                <textarea id="terms"></textarea>
            </div>

            <div class="full">
                <label>Delivery</label>
                <textarea id="delivery"></textarea>
            </div>

            <div>
                <label>Stock Location</label>
                <input id="stockLocation" placeholder="Enter manually if needed">
            </div>

            <div>
                <label>PI Number</label>
                <input id="piNumber">
            </div>

        </div>

        <button class="primary" onclick="generateMessage()">
            Generate Message
        </button>

    </div>


    <div class="card">
        <h2>3. Ready-to-Copy Message</h2>

        <textarea id="message" placeholder="Your generated message will appear here..."></textarea>

        <button class="primary" onclick="copyMessage()">
            Copy Message
        </button>
    </div>

</div>


<script>

pdfjsLib.GlobalWorkerOptions.workerSrc =
    "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js";


/* ---------------------------------------------------
   BASIC HELPERS
--------------------------------------------------- */

function clean(value) {
    if (!value) return "";

    return value
        .replace(/\u00a0/g, " ")
        .replace(/[ \t]+/g, " ")
        .replace(/\s+\n/g, "\n")
        .trim();
}


function cleanEmail(value) {
    if (!value) return "";

    return value
        .replace(/\s+/g, " ")
        .replace(/\s*\/\s*/g, " / ")
        .trim();
}


function setStatus(text) {
    document.getElementById("status").textContent = text;
}


/* ---------------------------------------------------
   PDF EXTRACTION
--------------------------------------------------- */

async function extractPDF(file) {

    const arrayBuffer = await file.arrayBuffer();

    const pdf = await pdfjsLib.getDocument({
        data: arrayBuffer
    }).promise;

    let allText = [];

    for (let pageNumber = 1; pageNumber <= pdf.numPages; pageNumber++) {

        const page = await pdf.getPage(pageNumber);

        const content = await page.getTextContent();

        /*
         Reconstruct text based on the Y coordinate.
         This is much better than simply joining every PDF
         text object with spaces.
        */

        const rows = {};

        content.items.forEach(item => {

            const y = Math.round(item.transform[5]);

            if (!rows[y]) {
                rows[y] = [];
            }

            rows[y].push({
                x: item.transform[4],
                text: item.str
            });

        });

        const sortedRows = Object.keys(rows)
            .sort((a, b) => Number(b) - Number(a));

        sortedRows.forEach(y => {

            const row = rows[y]
                .sort((a, b) => a.x - b.x)
                .map(item => item.text)
                .join(" ");

            if (row.trim()) {
                allText.push(row.trim());
            }

        });
    }

    return allText.join("\n");
}


/* ---------------------------------------------------
   DOCX EXTRACTION
--------------------------------------------------- */

async function extractDOCX(file) {

    const arrayBuffer = await file.arrayBuffer();

    const result = await mammoth.extractRawText({
        arrayBuffer: arrayBuffer
    });

    return result.value;
}


/* ---------------------------------------------------
   FIND SECTION
--------------------------------------------------- */

function sectionBetween(lines, startLabels, endLabels) {

    let start = -1;

    for (let i = 0; i < lines.length; i++) {

        const line = lines[i].trim().toLowerCase();

        if (startLabels.some(label =>
            line.startsWith(label.toLowerCase())
        )) {
            start = i;
            break;
        }
    }

    if (start === -1) return [];

    let end = lines.length;

    for (let i = start + 1; i < lines.length; i++) {

        const line = lines[i].trim().toLowerCase();

        if (endLabels.some(label =>
            line.startsWith(label.toLowerCase())
        )) {
            end = i;
            break;
        }
    }

    return lines.slice(start, end);
}


/* ---------------------------------------------------
   EMAIL EXTRACTION
--------------------------------------------------- */

function extractEmails(text) {

    const matches = text.match(
        /[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi
    );

    if (!matches) return "";

    return [...new Set(matches)].join(" / ");
}


/* ---------------------------------------------------
   PHONE EXTRACTION
--------------------------------------------------- */

function extractPhone(text) {

    const match = text.match(
        /(?:Telephone|Tel|Phone|Mobile)\s*:?\s*([+0-9][+0-9\s().-]{6,})/i
    );

    if (!match) return "";

    return match[1].trim();
}


/* ---------------------------------------------------
   CONTACT BLOCK PARSER
--------------------------------------------------- */

function parseContactSection(section) {

    const result = {
        name: "",
        address: "",
        attn: "",
        email: "",
        phone: ""
    };

    if (!section || section.length === 0) {
        return result;
    }

    const text = section.join("\n");

    const lines = section
        .map(x => x.trim())
        .filter(Boolean);


    /* NAME */

    const first = lines[0] || "";

    if (!/^(bill to|ship to|address|attn|email|telephone|tel|phone)/i.test(first)) {
        result.name = first.replace(/^(Bill To|Ship To)\s*:?\s*/i, "").trim();
    }


    /* ADDRESS */

    const addressIndex = lines.findIndex(x =>
        /^address\s*:/i.test(x)
    );

    if (addressIndex !== -1) {

        let address = lines[addressIndex]
            .replace(/^address\s*:\s*/i, "")
            .trim();

        for (let i = addressIndex + 1; i < lines.length; i++) {

            if (/^(attn|email|telephone|tel|phone)\s*:/i.test(lines[i])) {
                break;
            }

            address += " " + lines[i];
        }

        result.address = clean(address);
    }


    /* ATTN */

    const attnMatch = text.match(
        /(?:Attn|Attention)\s*:?\s*([^\n]+)/i
    );

    if (attnMatch) {
        result.attn = clean(attnMatch[1]);
    }


    /* EMAIL */

    const emailMatch = text.match(
        /Email\s*:?\s*([^\n]+)/i
    );

    if (emailMatch) {
        result.email = cleanEmail(emailMatch[1]);
    }

    const extractedEmails = extractEmails(text);

    if (extractedEmails) {
        result.email = extractedEmails;
    }


    /* PHONE */

    result.phone = extractPhone(text);

    return result;
}


/* ---------------------------------------------------
   PRODUCT PARSER
--------------------------------------------------- */

function parseProducts(lines) {

    const products = [];

    for (let i = 0; i < lines.length; i++) {

        const line = lines[i].trim();

        /*
          Looks for:

          description   quantity   unit price   total

          Example:

          Sennheiser Momentum 4 ... 700383 60 139 8340
        */

        const match = line.match(
            /^(.+?)\s+(\d+)\s+([\d,.]+)\s+([\d,.]+)$/
        );

        if (match) {

            const description = clean(match[1]);
            const quantity = match[2];
            const unitPrice = match[3];

            if (
                description.length > 5 &&
                !/^(total|subtotal|amount|quantity|price)/i.test(description)
            ) {

                products.push(
                    `${description} x${quantity}pcs @ $${unitPrice}`
                );

                continue;
            }
        }


        /*
          Some DOCX tables separate the fields.

          Example:

          description
          60
          139
          8340
        */

        if (
            i + 3 < lines.length &&
            /^\d+$/.test(lines[i + 1]) &&
            /^[\d,.]+$/.test(lines[i + 2]) &&
            /^[\d,.]+$/.test(lines[i + 3])
        ) {

            const description = clean(line);
            const quantity = lines[i + 1];
            const unitPrice = lines[i + 2];

            if (
                description.length > 5 &&
                !/^(total|subtotal|amount|quantity|price)/i.test(description)
            ) {

                products.push(
                    `${description} x${quantity}pcs @ $${unitPrice}`
                );
            }
        }
    }

    return [...new Set(products)];
}


/* ---------------------------------------------------
   MAIN PARSER
--------------------------------------------------- */

function parseInvoice(rawText) {

    const lines = rawText
        .split(/\r?\n/)
        .map(x => x.trim())
        .filter(Boolean);


    const result = {

        billTo: "",
        billAddress: "",
        billAttn: "",
        billEmail: "",
        billPhone: "",

        shipTo: "",
        shipAddress: "",
        shipAttn: "",
        shipEmail: "",
        shipPhone: "",

        products: "",
        terms: "",
        delivery: "",
        stockLocation: "",
        piNumber: ""
    };


    /* ------------------------------------------------
       PI NUMBER
    ------------------------------------------------ */

    const piMatch = rawText.match(
        /(?:PI\s*(?:No\.?|Number)?|Proforma Invoice\s*(?:No\.?|Number)?)\s*:?\s*([A-Z0-9-]+)/i
    );

    if (piMatch) {
        result.piNumber = piMatch[1].trim();
    }


    /* ------------------------------------------------
       BILL TO / SHIP TO
    ------------------------------------------------ */

    const billStart = lines.findIndex(x =>
        /^bill\s*to\s*:/i.test(x)
    );

    const shipStart = lines.findIndex(x =>
        /^ship\s*to\s*:/i.test(x)
    );


    if (billStart !== -1) {

        const billEnd =
            shipStart !== -1
                ? shipStart
                : Math.min(billStart + 20, lines.length);

        const billLines = lines.slice(billStart, billEnd);

        result.billTo = billLines[0]
            .replace(/^bill\s*to\s*:\s*/i, "")
            .trim();

        const bill = parseContactSection(billLines);

        if (bill.address) result.billAddress = bill.address;
        if (bill.attn) result.billAttn = bill.attn;
        if (bill.email) result.billEmail = bill.email;
        if (bill.phone) result.billPhone = bill.phone;
    }


    if (shipStart !== -1) {

        const shipEnd = Math.min(shipStart + 20, lines.length);

        const shipLines = lines.slice(shipStart, shipEnd);

        result.shipTo = shipLines[0]
            .replace(/^ship\s*to\s*:\s*/i, "")
            .trim();

        const ship = parseContactSection(shipLines);

        if (ship.address) result.shipAddress = ship.address;
        if (ship.attn) result.shipAttn = ship.attn;
        if (ship.email) result.shipEmail = ship.email;
        if (ship.phone) result.shipPhone = ship.phone;
    }


    /* ------------------------------------------------
       FALLBACK EMAIL / PHONE SEARCH
    ------------------------------------------------ */

    if (!result.billEmail || !result.shipEmail) {

        const allEmails = [
            ...new Set(
                rawText.match(
                    /[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi
                ) || []
            )
        ];

        if (!result.billEmail && allEmails.length) {
            result.billEmail = allEmails.slice(0, 4).join(" / ");
        }

        if (!result.shipEmail && allEmails.length > 4) {
            result.shipEmail = allEmails.slice(4).join(" / ");
        }
    }


    /* ------------------------------------------------
       PRODUCTS
    ------------------------------------------------ */

    const products = parseProducts(lines);

    result.products = products.join("\n");


    /* ------------------------------------------------
       TERMS
    ------------------------------------------------ */

    const termsMatch = rawText.match(
        /Terms\s*:?\s*([\s\S]*?)(?=\n(?:Delivery|Stock Location|Total|Bank|PI make|$))/i
    );

    if (termsMatch) {
        result.terms = clean(termsMatch[1]);
    }


    /* ------------------------------------------------
       DELIVERY
    ------------------------------------------------ */

    const deliveryMatch = rawText.match(
        /Delivery\s*:?\s*([\s\S]*?)(?=\n(?:Stock Location|PI make|Bank|$))/i
    );

    if (deliveryMatch) {
        result.delivery = clean(deliveryMatch[1]);
    }


    /*
      IMPORTANT:
      We deliberately DO NOT try to invent Stock Location.
      If the PI does not state it, it remains blank.
    */

    result.stockLocation = "";


    return result;
}


/* ---------------------------------------------------
   PUT DATA INTO FORM
--------------------------------------------------- */

function populateForm(data) {

    document.getElementById("billTo").value = data.billTo;
    document.getElementById("billAddress").value = data.billAddress;
    document.getElementById("billAttn").value = data.billAttn;
    document.getElementById("billEmail").value = data.billEmail;
    document.getElementById("billPhone").value = data.billPhone;

    document.getElementById("shipTo").value = data.shipTo;
    document.getElementById("shipAddress").value = data.shipAddress;
    document.getElementById("shipAttn").value = data.shipAttn;
    document.getElementById("shipEmail").value = data.shipEmail;
    document.getElementById("shipPhone").value = data.shipPhone;

    document.getElementById("products").value = data.products;

    document.getElementById("terms").value = data.terms;
    document.getElementById("delivery").value = data.delivery;

    document.getElementById("stockLocation").value =
        data.stockLocation;

    document.getElementById("piNumber").value =
        data.piNumber;
}


/* ---------------------------------------------------
   EXTRACT BUTTON
--------------------------------------------------- */

async function extractInvoice() {

    const file =
        document.getElementById("fileInput").files[0];

    if (!file) {

        setStatus("Please select a PDF or DOCX file first.");

        return;
    }


    try {

        setStatus("Reading invoice...");


        let rawText = "";


        if (file.name.toLowerCase().endsWith(".pdf")) {

            rawText = await extractPDF(file);

        } else if (
            file.name.toLowerCase().endsWith(".docx")
        ) {

            rawText = await extractDOCX(file);

        } else {

            throw new Error(
                "Please upload a PDF or DOCX file."
            );
        }


        if (!rawText.trim()) {

            throw new Error(
                "No readable text was found in this file."
            );
        }


        console.log("EXTRACTED TEXT:");
        console.log(rawText);


        const data = parseInvoice(rawText);

        populateForm(data);

        setStatus(
            "Invoice extracted. Please review the information before generating the message."
        );


        generateMessage();


    } catch (error) {

        console.error(error);

        setStatus(
            "Extraction error: " + error.message
        );
    }
}


/* ---------------------------------------------------
   GENERATE MESSAGE
--------------------------------------------------- */

function generateMessage() {

    const billTo =
        document.getElementById("billTo").value.trim();

    const billAddress =
        document.getElementById("billAddress").value.trim();

    const billAttn =
        document.getElementById("billAttn").value.trim();

    const billEmail =
        document.getElementById("billEmail").value.trim();

    const billPhone =
        document.getElementById("billPhone").value.trim();


    const shipTo =
        document.getElementById("shipTo").value.trim();

    const shipAddress =
        document.getElementById("shipAddress").value.trim();

    const shipAttn =
        document.getElementById("shipAttn").value.trim();

    const shipEmail =
        document.getElementById("shipEmail").value.trim();

    const shipPhone =
        document.getElementById("shipPhone").value.trim();


    const products =
        document.getElementById("products").value.trim();

    const terms =
        document.getElementById("terms").value.trim();

    const delivery =
        document.getElementById("delivery").value.trim();

    const stockLocation =
        document.getElementById("stockLocation").value.trim();

    const piNumber =
        document.getElementById("piNumber").value.trim();


    let output = "";


    output += `Bill To: ${billTo}\n`;
    output += `Address: ${billAddress}\n`;
    output += `Attn: ${billAttn}\n`;
    output += `Email: ${billEmail}\n`;
    output += `Telephone: ${billPhone}\n\n`;


    output += `Ship To: ${shipTo}\n`;
    output += `Address: ${shipAddress}\n`;
    output += `Attn: ${shipAttn}\n`;
    output += `Email: ${shipEmail}\n`;
    output += `Telephone: ${shipPhone}\n\n`;


    if (products) {
        output += products + "\n\n";
    }


    output += `Terms: ${terms}\n`;
    output += `Delivery: ${delivery}\n\n`;

    output += `Stock Location: ${stockLocation}\n`;

    output += `PI make by incharge person. Thanks \n`;

    output += piNumber;


    document.getElementById("message").value =
        output;
}


/* ---------------------------------------------------
   COPY
--------------------------------------------------- */

async function copyMessage() {

    const message =
        document.getElementById("message").value;

    if (!message) {
        return;
    }

    try {

        await navigator.clipboard.writeText(message);

        setStatus("Message copied!");

    } catch (error) {

        document.getElementById("message").select();

        document.execCommand("copy");

        setStatus("Message copied!");
    }
}

</script>

</body>
</html>            border-radius: 14px;
            padding: 28px;
            margin-bottom: 24px;
            box-shadow: 0 4px 18px rgba(0,0,0,0.06);
        }

        .card h2 {
            margin-top: 0;
            font-size: 21px;
        }

        .upload-area {
            border: 2px dashed #b8c0c8;
            border-radius: 12px;
            padding: 45px 20px;
            text-align: center;
            cursor: pointer;
        }

        .upload-area:hover {
            background: #fafafa;
            border-color: #333;
        }

        .upload-icon {
            font-size: 42px;
            margin-bottom: 12px;
        }

        .upload-area strong {
            display: block;
            font-size: 18px;
            margin-bottom: 8px;
        }

        .upload-area span {
            color: #777;
            font-size: 14px;
        }

        #fileInput {
            display: none;
        }

        .file-name {
            margin-top: 15px;
            text-align: center;
            font-size: 14px;
        }

        .button-row {
            display: flex;
            gap: 12px;
            margin-top: 20px;
            flex-wrap: wrap;
        }

        button {
            border: none;
            border-radius: 8px;
            padding: 13px 22px;
            font-size: 15px;
            cursor: pointer;
            font-weight: 600;
        }

        .primary-button {
            background: #222;
            color: white;
        }

        .primary-button:hover {
            background: #000;
        }

        .secondary-button {
            background: #e9edf1;
            color: #222;
        }

        .field-grid {
            display: grid;
            grid-template-columns: repeat(2, 1fr);
            gap: 18px;
        }

        .field {
            display: flex;
            flex-direction: column;
        }

        .field.full {
            grid-column: 1 / -1;
        }

        label {
            font-size: 13px;
            font-weight: 700;
            margin-bottom: 7px;
            color: #444;
        }

        input,
        textarea {
            width: 100%;
            border: 1px solid #d5d9dd;
            border-radius: 7px;
            padding: 11px 12px;
            font-size: 14px;
            font-family: inherit;
        }

        textarea {
            min-height: 80px;
            resize: vertical;
        }

        input:focus,
        textarea:focus {
            outline: none;
            border-color: #555;
        }

        .message-box {
            width: 100%;
            min-height: 430px;
            background: #f8f9fa;
            border: 1px solid #d9dde1;
            border-radius: 10px;
            padding: 18px;
            font-size: 14px;
            line-height: 1.6;
            white-space: pre-wrap;
        }

        .status {
            margin-top: 15px;
            font-size: 14px;
            color: #666;
        }

        .success {
            color: #188038;
        }

        .error {
            color: #c5221f;
        }

        @media (max-width: 700px) {

            .field-grid {
                grid-template-columns: 1fr;
            }

            .field.full {
                grid-column: auto;
            }

        }

    </style>
</head>

<body>

<div class="container">

    <div class="header">

        <h1>PI Message Generator</h1>

        <p>
            Upload a Proforma Invoice and generate a ready-to-copy
            supply chain message.
        </p>

    </div>


    <!-- ========================= -->
    <!-- UPLOAD -->
    <!-- ========================= -->

    <div class="card">

        <h2>1. Upload Proforma Invoice</h2>

        <label for="fileInput" class="upload-area">

            <div class="upload-icon">📄</div>

            <strong>Drop your PDF or DOCX here</strong>

            <span>
                or click to browse files
            </span>

        </label>

        <input
            type="file"
            id="fileInput"
            accept=".pdf,.docx"
        >

        <div
            id="fileName"
            class="file-name"
        >
            No file selected
        </div>

        <div class="button-row">

            <button
                class="primary-button"
                id="extractButton"
            >
                Extract Invoice
            </button>

        </div>

        <div
            id="status"
            class="status"
        ></div>

    </div>


    <!-- ========================= -->
    <!-- REVIEW -->
    <!-- ========================= -->

    <div class="card">

        <h2>2. Review / Edit Information</h2>

        <div class="field-grid">

            <div class="field full">
                <label>Bill To</label>
                <input id="billTo">
            </div>

            <div class="field full">
                <label>Bill To Address</label>
                <textarea id="billAddress"></textarea>
            </div>

            <div class="field">
                <label>Bill To Attn</label>
                <input id="billAttn">
            </div>

            <div class="field">
                <label>Bill To Email</label>
                <input id="billEmail">
            </div>

            <div class="field">
                <label>Bill To Telephone</label>
                <input id="billTelephone">
            </div>


            <div class="field full">
                <label>Ship To</label>
                <input id="shipTo">
            </div>

            <div class="field full">
                <label>Ship To Address</label>
                <textarea id="shipAddress"></textarea>
            </div>

            <div class="field">
                <label>Ship To Attn</label>
                <input id="shipAttn">
            </div>

            <div class="field">
                <label>Ship To Email</label>
                <input id="shipEmail">
            </div>

            <div class="field">
                <label>Ship To Telephone</label>
                <input id="shipTelephone">
            </div>


            <div class="field full">
                <label>Products</label>

                <textarea
                    id="products"
                    placeholder="Product x quantity @ unit price"
                ></textarea>

            </div>


            <div class="field full">

                <label>Terms</label>

                <textarea id="terms"></textarea>

            </div>


            <div class="field full">

                <label>Delivery</label>

                <textarea id="delivery"></textarea>

            </div>


            <div class="field">

                <label>Stock Location</label>

                <input
                    id="stockLocation"
                    placeholder="Leave blank if not stated in PI"
                >

            </div>


            <div class="field">

                <label>PI Number</label>

                <input id="piNumber">

            </div>

        </div>


        <div class="button-row">

            <button
                class="primary-button"
                id="generateButton"
            >
                Generate Message
            </button>

            <button
                class="secondary-button"
                id="clearButton"
            >
                Clear
            </button>

        </div>

    </div>


    <!-- ========================= -->
    <!-- OUTPUT -->
    <!-- ========================= -->

    <div class="card">

        <h2>3. Ready-to-Copy Message</h2>

        <div
            id="messageBox"
            class="message-box"
        >
Your generated message will appear here.
        </div>

        <div class="button-row">

            <button
                class="primary-button"
                id="copyButton"
            >
                Copy Message
            </button>

        </div>

        <div
            id="copyStatus"
            class="status"
        ></div>

    </div>

</div>


<script type="module">

    /*
     * PDF.js
     */

    import * as pdfjsLib from
        "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.4.168/pdf.min.mjs";


    /*
     * PDF worker
     */

    pdfjsLib.GlobalWorkerOptions.workerSrc =
        "https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.4.168/pdf.worker.min.mjs";


    /*
     * Elements
     */

    const fileInput =
        document.getElementById("fileInput");

    const fileName =
        document.getElementById("fileName");

    const extractButton =
        document.getElementById("extractButton");

    const generateButton =
        document.getElementById("generateButton");

    const clearButton =
        document.getElementById("clearButton");

    const copyButton =
        document.getElementById("copyButton");

    const status =
        document.getElementById("status");

    const messageBox =
        document.getElementById("messageBox");

    const copyStatus =
        document.getElementById("copyStatus");


    /*
     * File selection
     */

    fileInput.addEventListener(
        "change",
        function () {

            if (fileInput.files.length === 0) {

                fileName.textContent =
                    "No file selected";

                return;

            }

            fileName.textContent =
                "Selected file: " +
                fileInput.files[0].name;

            status.textContent =
                "";

        }
    );


    /*
     * Extract PDF text
     */

    async function extractPDF(file) {

        const arrayBuffer =
            await file.arrayBuffer();

        const pdf =
            await pdfjsLib.getDocument({
                data: arrayBuffer
            }).promise;

        let fullText = "";


        for (
            let pageNumber = 1;
            pageNumber <= pdf.numPages;
            pageNumber++
        ) {

            const page =
                await pdf.getPage(pageNumber);

            const textContent =
                await page.getTextContent();

            const pageText =
                textContent.items
                    .map(item => item.str)
                    .join(" ");

            fullText +=
                pageText + "\n";

        }

        return fullText;

    }


    /*
     * Extract DOCX text
     */

    async function extractDOCX(file) {

        const arrayBuffer =
            await file.arrayBuffer();

        const result =
            await mammoth.extractRawText({
                arrayBuffer: arrayBuffer
            });

        return result.value;

    }


    /*
     * Extract file
     */

    async function extractFile(file) {

        const fileName =
            file.name.toLowerCase();


        if (fileName.endsWith(".pdf")) {

            return await extractPDF(file);

        }


        if (fileName.endsWith(".docx")) {

            return await extractDOCX(file);

        }


        throw new Error(
            "Only PDF and DOCX files are supported."
        );

    }


    /*
     * Find text between labels
     */

    function textBetween(
        text,
        startLabel,
        endLabels
    ) {

        const start =
            text.toLowerCase().indexOf(
                startLabel.toLowerCase()
            );

        if (start === -1) {
            return "";
        }


        let value =
            text.substring(
                start + startLabel.length
            );


        let endPosition =
            value.length;


        for (
            const endLabel of endLabels
        ) {

            const position =
                value.toLowerCase().indexOf(
                    endLabel.toLowerCase()
                );

            if (
                position !== -1 &&
                position < endPosition
            ) {

                endPosition =
                    position;

            }

        }


        return value
            .substring(0, endPosition)
            .replace(/\s+/g, " ")
            .trim();

    }


    /*
     * Extract invoice number
     */

    function extractPI(text) {

        const match =
            text.match(
                /PI\s*(?:No\.?|Number)?\s*[:#]?\s*([A-Z0-9-]+)/i
            );

        return match
            ? match[1]
            : "";

    }


    /*
     * Extract email addresses
     */

    function extractEmails(text) {

        const emails =
            text.match(
                /[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}/gi
            );

        if (!emails) {
            return "";
        }

        return [
            ...new Set(emails)
        ].join(" / ");

    }


    /*
     * Extract telephone numbers
     */

    function extractPhone(text) {

        const matches =
            text.match(
                /\+?\d[\d\s().-]{6,}\d/g
            );

        if (!matches) {
            return "";
        }

        return [
            ...new Set(
                matches.map(
                    x => x.trim()
                )
            )
        ].join(" / ");

    }


    /*
     * Extract products
     *
     * First version:
     * Detect lines containing quantities
     * and prices.
     */

    function extractProducts(text) {

        const lines =
            text
                .split(/\n/)
                .map(
                    line => line
                        .replace(/\s+/g, " ")
                        .trim()
                )
                .filter(Boolean);


        const products = [];


        for (
            let i = 0;
            i < lines.length;
            i++
        ) {

            const line =
                lines[i];


            /*
             * Detect:
             *
             * description
             * quantity
             * unit price
             */

            const match =
                line.match(
                    /(.+?)\s+(\d+)\s+([\d,]+\.\d{2})\s+(?:[\d,]+\.\d{2})$/
                );


            if (match) {

                const description =
                    match[1].trim();

                const quantity =
                    match[2];

                const price =
                    match[3].replace(
                        /,/g,
                        ""
                    );


                products.push(
                    `${description} x${quantity}pcs @ $${Number(price)}`
                );

            }

        }


        return products.join("\n");

    }


    /*
     * Parse invoice
     */

    function parseInvoice(text) {

        const normalized =
            text
                .replace(/\r/g, "\n")
                .replace(/[ \t]+/g, " ");


        /*
         * Bill To
         */

        const billToBlock =
            textBetween(
                normalized,
                "Bill To:",
                [
                    "PI No:",
                    "Date:",
                    "Ship To:"
                ]
            );


        /*
         * Ship To
         */

        const shipToBlock =
            textBetween(
                normalized,
                "Ship To:",
                [
                    "Delivery:",
                    "Terms:",
                    "Bank Details"
                ]
            );


        /*
         * Customer name
         */

        let billTo =
            billToBlock;

        billTo =
            billTo
                .replace(
                    /Address:.*/i,
                    ""
                )
                .trim();


        /*
         * Bill address
         */

        let billAddress =
            textBetween(
                billToBlock,
                "Address:",
                [
                    "Attn:",
                    "Email:",
                    "Telephone:"
                ]
            );


        /*
         * Bill Attn
         */

        let billAttn =
            textBetween(
                billToBlock,
                "Attn:",
                [
                    "Email:",
                    "Telephone:"
                ]
            );


        /*
         * Ship customer
         */

        let shipTo =
            shipToBlock
                .replace(
                    /Address:.*/i,
                    ""
                )
                .trim();


        /*
         * Ship address
         */

        let shipAddress =
            textBetween(
                shipToBlock,
                "Address:",
                [
                    "Attn:",
                    "Email:",
                    "Telephone:",
                    "Delivery:"
                ]
            );


        /*
         * Ship Attn
         */

        let shipAttn =
            textBetween(
                shipToBlock,
                "Attn:",
                [
                    "Email:",
                    "Telephone:"
                ]
            );


        /*
         * Terms
         */

        let terms =
            textBetween(
                normalized,
                "Terms:",
                [
                    "Delivery:",
                    "Bank Details"
                ]
            );


        /*
         * Delivery
         */

        let delivery =
            textBetween(
                normalized,
                "Delivery:",
                [
                    "Bank Details",
                    "Remark:"
                ]
            );


        /*
         * Emails
         */

        const allEmails =
            extractEmails(text);


        /*
         * Phones
         */

        const allPhones =
            extractPhone(text);


        /*
         * Products
         */

        const products =
            extractProducts(text);


        /*
         * PI
         */

        const piNumber =
            extractPI(text);


        return {

            billTo,
            billAddress,
            billAttn,

            billEmail:
                allEmails,

            billTelephone:
                allPhones,

            shipTo,
            shipAddress,
            shipAttn,

            shipEmail:
                allEmails,

            shipTelephone:
                allPhones,

            products,
            terms,
            delivery,

            stockLocation:
                "",

            piNumber

        };

    }


    /*
     * Put extracted information
     * into the form.
     */

    function populateForm(data) {

        document.getElementById(
            "billTo"
        ).value =
            data.billTo || "";


        document.getElementById(
            "billAddress"
        ).value =
            data.billAddress || "";


        document.getElementById(
            "billAttn"
        ).value =
            data.billAttn || "";


        document.getElementById(
            "billEmail"
        ).value =
            data.billEmail || "";


        document.getElementById(
            "billTelephone"
        ).value =
            data.billTelephone || "";


        document.getElementById(
            "shipTo"
        ).value =
            data.shipTo || "";


        document.getElementById(
            "shipAddress"
        ).value =
            data.shipAddress || "";


        document.getElementById(
            "shipAttn"
        ).value =
            data.shipAttn || "";


        document.getElementById(
            "shipEmail"
        ).value =
            data.shipEmail || "";


        document.getElementById(
            "shipTelephone"
        ).value =
            data.shipTelephone || "";


        document.getElementById(
            "products"
        ).value =
            data.products || "";


        document.getElementById(
            "terms"
        ).value =
            data.terms || "";


        document.getElementById(
            "delivery"
        ).value =
            data.delivery || "";


        /*
         * IMPORTANT:
         *
         * Stock location is deliberately
         * left blank.
         */

        document.getElementById(
            "stockLocation"
        ).value =
            "";


        document.getElementById(
            "piNumber"
        ).value =
            data.piNumber || "";

    }


    /*
     * Extract button
     */

    extractButton.addEventListener(
        "click",
        async function () {

            if (
                fileInput.files.length === 0
            ) {

                status.textContent =
                    "Please select a PDF or DOCX file first.";

                status.className =
                    "status error";

                return;

            }


            const file =
                fileInput.files[0];


            try {

                status.textContent =
                    "Reading invoice...";

                status.className =
                    "status";


                const text =
                    await extractFile(file);


                if (!text.trim()) {

                    throw new Error(
                        "No readable text was found in this file."
                    );

                }


                const data =
                    parseInvoice(text);


                populateForm(data);


                status.textContent =
                    "Invoice extracted. Please review the information before generating the message.";

                status.className =
                    "status success";


                console.log(
                    "Extracted invoice text:",
                    text
                );


            } catch (error) {

                console.error(error);

                status.textContent =
                    "Extraction failed: " +
                    error.message;

                status.className =
                    "status error";

            }

        }
    );


    /*
     * Generate message
     */

    generateButton.addEventListener(
        "click",
        function () {

            const billTo =
                document.getElementById(
                    "billTo"
                ).value.trim();


            const billAddress =
                document.getElementById(
                    "billAddress"
                ).value.trim();


            const billAttn =
                document.getElementById(
                    "billAttn"
                ).value.trim();


            const billEmail =
                document.getElementById(
                    "billEmail"
                ).value.trim();


            const billTelephone =
                document.getElementById(
                    "billTelephone"
                ).value.trim();


            const shipTo =
                document.getElementById(
                    "shipTo"
                ).value.trim();


            const shipAddress =
                document.getElementById(
                    "shipAddress"
                ).value.trim();


            const shipAttn =
                document.getElementById(
                    "shipAttn"
                ).value.trim();


            const shipEmail =
                document.getElementById(
                    "shipEmail"
                ).value.trim();


            const shipTelephone =
                document.getElementById(
                    "shipTelephone"
                ).value.trim();


            const products =
                document.getElementById(
                    "products"
                ).value.trim();


            const terms =
                document.getElementById(
                    "terms"
                ).value.trim();


            const delivery =
                document.getElementById(
                    "delivery"
                ).value.trim();


            const stockLocation =
                document.getElementById(
                    "stockLocation"
                ).value.trim();


            const piNumber =
                document.getElementById(
                    "piNumber"
                ).value.trim();


            const message =

`Bill To: ${billTo}
Address: ${billAddress}
Attn: ${billAttn}
Email: ${billEmail}
Telephone: ${billTelephone}

Ship To: ${shipTo}
Address: ${shipAddress}
Attn: ${shipAttn}
Email: ${shipEmail}
Telephone: ${shipTelephone}

${products}

Terms: ${terms}
Delivery: ${delivery}

Stock Location: ${stockLocation}
PI make by incharge person. Thanks
${piNumber}`;


            messageBox.textContent =
                message;

        }
    );


    /*
     * Copy button
     */

    copyButton.addEventListener(
        "click",
        async function () {

            const message =
                messageBox.textContent.trim();


            if (
                !message ||
                message ===
                "Your generated message will appear here."
            ) {

                copyStatus.textContent =
                    "There is no message to copy yet.";

                return;

            }


            try {

                await navigator.clipboard.writeText(
                    message
                );

                copyStatus.textContent =
                    "Message copied successfully!";

                copyStatus.className =
                    "status success";

            } catch (error) {

                copyStatus.textContent =
                    "Copy failed. Please select the message manually.";

                copyStatus.className =
                    "status error";

            }

        }
    );


    /*
     * Clear
     */

    clearButton.addEventListener(
        "click",
        function () {

            document
                .querySelectorAll(
                    "input, textarea"
                )
                .forEach(
                    element => {
                        element.value = "";
                    }
                );


            fileInput.value =
                "";


            fileName.textContent =
                "No file selected";


            messageBox.textContent =
                "Your generated message will appear here.";


            status.textContent =
                "";


            copyStatus.textContent =
                "";

        }
    );

</script>

</body>
</html>
