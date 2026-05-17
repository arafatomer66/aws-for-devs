# Textract — OCR + Forms + Tables

**TL;DR** — OCR with structure. Pulls text, key-value pairs, tables, and signatures from scanned docs and PDFs. Better than Rekognition for documents.

## What it does

- **DetectDocumentText** — plain OCR.
- **AnalyzeDocument** — forms (key-value) + tables + signatures.
- **AnalyzeID** — driver licenses, passports.
- **AnalyzeExpense** — invoices, receipts.
- **AnalyzeLending** — mortgage documents.
- **Queries** — ask "What is the invoice number?" against a doc.

## Real-world example

> ShareDeal stores supplier invoices:
> - Vendor sends PDF invoice via email → Lambda → Textract `AnalyzeExpense`.
> - Extracts vendor, total, line items, tax automatically.
> - Pushes structured data to Aurora.

## Usage

```js
import { TextractClient, AnalyzeExpenseCommand } from "@aws-sdk/client-textract";
const tx = new TextractClient({ region: "ap-south-1" });

const res = await tx.send(new AnalyzeExpenseCommand({
  Document: { S3Object: { Bucket: "invoices", Name: "inv-001.pdf" } },
}));
console.log(res.ExpenseDocuments);
```

For multi-page PDFs, use async `StartDocumentTextDetection` + `GetDocumentTextDetection`.

## Pricing

- **DetectDocumentText:** $1.50 per 1k pages.
- **AnalyzeDocument:** $15 per 1k pages for forms+tables.
- **AnalyzeExpense / ID / Lending:** $10-50 per 1k.

## Gotchas

- **Async APIs** are required for big PDFs.
- **Confidence scores** vary; pair with human-in-the-loop for high-stakes data.
- **Some doc types not supported** — handwritten with caveats.

## Related

- [Rekognition](./rekognition.md) — for image labels, not docs
- [Comprehend](./comprehend.md) — for post-extraction NLP
