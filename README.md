# Accounting Document Extractor (SAP S/4HANA - FI Module)

This project was developed in an SAP S/4HANA environment using ABAP language in Eclipse (ADT).

## Project Objective
To fulfill a technical specification for extracting accounting document data (General Ledger accounts) directly from the SAP database to provide a clean data structure (CSV format) for a corporate legacy system.

## Technical Features & Business Rules
- **Tables Used:** Data selection via `INNER JOIN` between tables `BKPF` (Accounting Document Header) and `BSEG` (Accounting Document Segment/Line Items).
- **Applied Filters:** Filtering by Company Code (`BUKRS`), Document Number (`BELNR`), and Mandatory Fiscal Year (`GJAHR`).
- **Document Type Constraint:** Strict restriction to **SA** accounting document types (`BLART = 'SA'`).
- **Data Formatting & Cleansing:**
  - Mapping technical German Debit/Credit indicators (`S`/`H`) to the standard format (`D`/`C`).
  - Formatting date fields to the technical legacy pattern `YYYYMMDD`.
  - Handling and adapting decimal notation for currency fields.
- **User Interface (UI):** Implementation of translated text elements and integration with the native Windows file dialog (`CL_GUI_FRONTEND_SERVICES=>FILE_SAVE_DIALOG`) to allow the user to choose the destination folder and file name dynamically.

## ABAP Text Elements (Internationalization)
To support translation and avoid hardcoded strings, the following Text Symbols were created within the program:

| Symbol | Text (English Reference) | Context / Usage |
| :--- | :--- | :--- |
| **001** | Selection Filters: SAP -> Legacy System | Selection Screen Block Title |
| **002** | Invalid Fiscal Year. Please enter a value between 1900 and 2100. | Error Message (E) |
| **003** | No SA document types found for the specified filters. | Error Message (E) |
| **004** | --- OUTPUT CSV FILE PREVIEW --- | ALV / Write Preview Header |
| **005** | CompanyCode;FiscalYear;DocNumber;PostingDate;Currency;LineItem;GLAccount;PostingKey;DebitCredit;Amount| CSV File Header Line |
| **006** | Save CSV File | Windows File Dialog Title |
| **007** | CSV File successfully generated at: | Success Message |
