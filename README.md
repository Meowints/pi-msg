<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">

    <title>PI Message Generator</title>

    <!-- PDF.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/4.4.168/pdf.min.mjs"
        type="module"></script>

    <!-- Mammoth.js -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.8.0/mammoth.browser.min.js"></script>

    <style>

        * {
            box-sizing: border-box;
        }

        body {
            margin: 0;
            font-family: Arial, Helvetica, sans-serif;
            background: #f4f6f8;
            color: #222;
        }

        .container {
            max-width: 1100px;
            margin: auto;
            padding: 40px 20px;
        }

        .header {
            text-align: center;
            margin-bottom: 35px;
        }

        .header h1 {
            margin: 0 0 10px;
            font-size: 34px;
        }

        .header p {
            margin: 0;
            color: #666;
        }

        .card {
            background: white;
            border-radius: 14px;
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
