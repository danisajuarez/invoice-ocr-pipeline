# Invoice OCR Pipeline

A production pipeline built as part of my role at **Gcom Tecnología**. It turns messy supplier invoices — both digital PDFs and scanned images — into structured, validated data and imports approved results directly into an ERP. **Running in production for 6 clients.**

Built with the Claude Vision API as the extraction engine, wrapped in validation and retry logic to handle real-world invoice mess.

---

## The problem

Invoices arrive in every shape: clean digital PDFs from some suppliers, blurry phone photos and scans from others. Before this, someone keyed each one into the ERP by hand — slow, tedious, and error-prone, especially across six clients with different supplier bases.

The goal was to remove the manual keying entirely while keeping the data trustworthy enough to write straight into a production accounting system.

---

## Running in production

This isn't a demo or a proof of concept. The pipeline processes real invoices for **6 live clients**, feeding the data into the ERP they use to run their business. That constraint shaped every decision below: the bar wasn't "extracts text correctly most of the time," it was "safe to write into accounting without a human double-checking every row."

---

## Handling the messy cases

The hard part of invoice extraction isn't the clean PDFs — it's everything else. The pipeline assumes the input is unreliable and defends against it:

- **Field-level validation against rules.** Key fields — CUIT (tax ID), total, date — are validated against format and consistency rules before anything is trusted. A malformed CUIT or an impossible date doesn't get written; it gets caught.
- **Reprocess on failure.** When extraction comes back doubtful or a field fails validation, the invoice is retried with a different prompt rather than silently passing bad data downstream. Mixed-quality input (a sharp PDF vs. a crooked scan) is handled by the same flow instead of needing a separate path.

The design principle: **nothing dubious reaches the ERP unvalidated.** It's better to flag an invoice than to import a wrong total into a client's books.

---

## End-to-end ERP integration

The pipeline doesn't stop at "here's some JSON." Extracted, validated data is imported **directly into the ERP database (SIGE)** — closing the loop from raw invoice to a usable record inside the system the client already works in. No intermediate export, no manual upload step.

```
Invoice (PDF or scan)
        │
        ▼
  Claude Vision  ──►  structured extraction
        │
        ▼
   Validation  ──►  fail ──► reprocess with alt prompt
        │ pass
        ▼
   Direct import into SIGE (ERP database)
```

---

## Prompt design

The extraction layer uses Claude Vision with structured-output prompting tuned for invoices: the model is asked to return data in the exact shape the ERP import expects, and the reprocess path swaps in an alternative prompt when the first pass doesn't validate. Prompt design here is in service of the validation layer — the prompts and the rules were tuned together against real failing invoices, not in isolation.

---

## What I owned

Working under technical leadership at Gcom Tecnología, I was responsible for the extraction flow, structured-output prompts, tax and field validation, retry strategy, review experience and integration with the existing SIGE ERP.

The public repository documents the architecture and decisions without exposing client source code, invoices or credentials.

---

## Stack

- **Extraction:** Claude Vision API (Anthropic)
- **Backend:** Laravel / PHP
- **Target system:** SIGE ERP (direct database import)
- **Input formats:** digital PDFs + scanned images

---

*Professional project delivered at Gcom Tecnología as part of a broader ERP and e-commerce ecosystem serving multiple clients.*

[View the visual case study](https://danisajuarez.netlify.app/projects/ocr)
