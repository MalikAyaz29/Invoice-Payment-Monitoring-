# ⚡ Invoice & Payment Monitoring Automation

<p align="center">
  <img src="https://img.shields.io/badge/Platform-n8n-EA4B71?style=for-the-badge&logo=n8n&logoColor=white" alt="n8n" />
  <img src="https://img.shields.io/badge/Workflow-Reconciliation-3B82F6?style=for-the-badge" alt="Reconciliation" />
  <img src="https://img.shields.io/badge/Status-Ready%20to%20Deploy-10B981?style=for-the-badge" alt="Status" />
  <img src="https://img.shields.io/badge/License-MIT-6B7280?style=for-the-badge" alt="License" />
</p>

<p align="center">
  A lightweight, real-time n8n automation workflow that validates incoming payment transactions, reconciles amounts against invoices, and flags anomalies before downstream settlement.
</p>

---

## 🏗️ Architecture & Flow

The workflow operates as an event-driven microservice listening to payment gateway events and returning immediate verification results.

```mermaid
flowchart TD
    subgraph Ingestion["1. Ingestion"]
        A["🌐 Payment Event\n(POST Webhook)"]
    end

    subgraph Processing["2. Verification & Reconciliation Engine"]
        B["🔍 Schema & Value Validation"]
        C{"Amount Comparison"}
        D["🟢 PAID\n(Exact Match)"]
        E["🟡 PARTIALLY_PAID\n(Underpaid)"]
        F["🔴 OVERPAID\n(Requires Review)"]
        G["⚠️ Risk Detection\n• Duplicates\n• Unverified Refs"]
    end

    subgraph Response["3. Synchronous Response"]
        H["📤 JSON Outcome\n(Status, Difference & Risk Flags)"]
    end

    A --> B
    B --> C
    C -->|Paid == Expected| D
    C -->|Paid < Expected| E
    C -->|Paid > Expected| F
    D --> G
    E --> G
    F --> G
    G --> H

    classDef default fill:#1e293b,stroke:#3b82f6,stroke-width:1.5px,color:#f8fafc;
    classDef success fill:#064e3b,stroke:#10b981,stroke-width:2px,color:#ecfdf5;
    classDef warn fill:#78350f,stroke:#f59e0b,stroke-width:2px,color:#fffbeb;
    classDef alert fill:#7f1d1d,stroke:#ef4444,stroke-width:2px,color:#fef2f2;
    class D success;
    class E warn;
    class F,G alert;
```

---

## 🔄 How the Flow Works

| Step | Stage | What Happens |
| :---: | :--- | :--- |
| **1** | **Ingest** | Listens for POST events from payment providers via `/portfolio-payment-monitor`. |
| **2** | **Validate** | Ensures required fields (`invoiceId`, `paymentId`, amounts) are present and structurally valid. |
| **3** | **Reconcile** | Compares `expectedAmount` vs `paidAmount` to evaluate financial accuracy. |
| **4** | **Flag Risks** | Inspects flags for upstream duplicate markers, unverified references, or amount discrepancies. |
| **5** | **Respond** | Returns instant reconciliation verdict and marks if manual finance review is required. |

---

## 📊 Status Matrix

| Status | Trigger Condition | Manual Review? |
| :--- | :--- | :---: |
| `PAID` | Amount matches invoice balance | ❌ No |
| `PARTIALLY_PAID` | Paid amount is less than invoice total | ⚠️ Yes |
| `OVERPAID` | Paid amount exceeds invoice total | ⚠️ Yes |
| `UNPAID` | Payment amount is 0 | ❌ No |
| `INVALID_DATA` | Missing or negative amount values | ⚠️ Yes |

---

## ⚡ Quick Start

1. **Import Workflow:** Open **n8n** ➜ Click **Add Workflow** ➜ Select **Import from File** ➜ Choose [`Invoice_Payment_Monitoring.json`](./Invoice_Payment_Monitoring.json).
2. **Activate:** Toggle workflow status to **Active** to obtain your production webhook URL.
3. **Send Payload:** Point your payment gateway or webhook dispatcher to the generated URL:

```json
{
  "invoiceId": "INV-1001",
  "paymentId": "PAY-5001",
  "expectedAmount": 1500.00,
  "paidAmount": 1500.00,
  "currency": "USD"
}
```

---

## 📄 License

Distributed under the MIT License.
