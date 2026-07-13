# Week 4 – Azure Cloud Fundamentals & Data Pipeline Implementation using ADF

**Celebal Technologies Internship — Data Engineering / Cloud Track**
**Week 4 Assignment**

---

## 📌 Overview

This repository documents the completion of Week 4 of the Celebal Technologies internship, focused on Microsoft Azure cloud fundamentals and building a complete, end-to-end data pipeline using **Azure Storage Account** and **Azure Data Factory (ADF)**.

The goal was to move beyond theoretical cloud concepts and get hands-on experience provisioning real Azure resources, connecting them securely, and orchestrating a data movement pipeline — from raw CSV ingestion in Blob Storage to a validated, automated copy operation using ADF.

---

## 🎯 Objective

To understand Azure cloud concepts and build a complete, production-style data pipeline by:

- Exploring the Azure Portal and provisioning foundational resources
- Creating a Storage Account with Blob Containers to hold source data
- Building an Azure Data Factory instance to orchestrate data movement
- Configuring Linked Services and Datasets to connect ADF to Blob Storage
- Using the **Get Metadata** activity to validate file existence and properties before processing
- Using the **Copy Data** activity to move data from a source to a destination container
- Assigning proper **IAM roles** (least-privilege access) between ADF and Storage
- Executing and monitoring the pipeline end-to-end

---

## 🛠️ Tech Stack / Services Used

| Service | Purpose |
|---|---|
| **Azure Resource Group** | Logical container to group and manage all related resources |
| **Azure Storage Account (Blob Storage)** | Hosts source and destination containers for CSV data |
| **Azure Data Factory (V2)** | Orchestrates the data pipeline (ETL/ELT) |
| **ADF Linked Service** | Secure connection between ADF and the Storage Account |
| **ADF Datasets** | Define the structure and location of source/destination data |
| **Get Metadata Activity** | Validates file existence, size, and last-modified timestamp |
| **Copy Data Activity** | Copies data from source Blob container to destination Blob container |
| **Azure IAM (RBAC)** | Grants Data Factory's managed identity least-privilege access to Storage |

---

## 🏗️ Architecture

```
┌─────────────────────┐        ┌──────────────────────┐        ┌──────────────────────────┐
│   Blob Container      │        │   Azure Data Factory    │        │   Blob Container            │
│   "source-data"       │──────▶ │   pl_week4_pipeline      │──────▶ │   "destination-data"        │
│   (sample_data.csv)   │  read  │  Get Metadata → Copy Data│  write │  (copied CSV output)        │
└─────────────────────┘        └──────────────────────┘        └──────────────────────────┘
                                          │
                                          ▼
                                 IAM Role: Storage Blob
                                 Data Contributor (RBAC)
```

---

## 📂 Repository Structure

```
week4-azure-adf-pipeline/
│
├── README.md                     # This file
├── screenshots/
│   ├── 01_resource_group.png
│   ├── 02_storage_account.png
│   ├── 03_container_uploaded_file.png
│   ├── 04_data_factory_overview.png
│   ├── 05_adf_studio_home.png
│   ├── 06_linked_service.png
│   ├── 07_dataset_source.png
│   ├── 08_dataset_destination.png
│   ├── 09_get_metadata_output.png
│   ├── 10_pipeline_canvas.png
│   ├── 11_pipeline_run_succeeded.png
│   ├── 12_iam_role_assignment.png
│   └── 13_destination_container_final.png
├── sample-data/
│   └── sample_data.csv           # Sample source dataset used in the pipeline
└── docs/
    └── Week4_Azure_ADF_Report.docx  # Full detailed report (Word format)
```

---

## 🚀 Step-by-Step Implementation

### 1️⃣ Explore Azure Portal & Create a Resource Group

- Logged into the Azure Portal using an Azure for Students subscription
- Created a Resource Group: **`rg-week4-datapipeline`**
- Region: **Central India** (chosen after resolving a subscription region-restriction error — see [Issues & Fixes](#-issues--fixes) below)

📸 `screenshots/01_resource_group.png`

---

### 2️⃣ Storage Setup

- Created a Storage Account: **`sumitweek4pipe2706`** (Standard performance, LRS redundancy, general-purpose v2)
- Created two Blob Containers:
  - `source-data` — holds the input CSV file
  - `destination-data` — receives the copied output file
- Uploaded a sample employee dataset (`sample_data.csv`) to `source-data`

📸 `screenshots/02_storage_account.png`
📸 `screenshots/03_container_uploaded_file.png`

---

### 3️⃣ ADF Basics

- Created an Azure Data Factory (V2): **`adf-sumitweek4-2706`**, same resource group and region as the storage account
- Explored the ADF Studio UI — **Author**, **Monitor**, and **Manage** panes
- Created a **Linked Service** (`ls_blobstorage` / `AzureBlobStoragesumit`) connecting ADF to the Storage Account via Account Key authentication, validated with **Test connection**
- Created two **Datasets**:
  - `ds_source_csv` — points to the CSV file in `source-data`
  - `ds_destination_csv` — points to the `destination-data` container
- Added a **Get Metadata** activity to retrieve `Item name`, `Size`, and `Last modified` for the source file

📸 `screenshots/04_data_factory_overview.png`
📸 `screenshots/05_adf_studio_home.png`
📸 `screenshots/06_linked_service.png`
📸 `screenshots/07_dataset_source.png`
📸 `screenshots/08_dataset_destination.png`
📸 `screenshots/09_get_metadata_output.png`

---

### 4️⃣ Pipeline Development

- Built pipeline **`pl_week4_pipeline`**
- Added a **Copy Data** activity, chained after Get Metadata using an "On Success" dependency
- Configured:
  - **Source**: `ds_source_csv`
  - **Sink**: `ds_destination_csv`
- (ForEach activity considered but not required, since only a single file was processed)

📸 `screenshots/10_pipeline_canvas.png`

---

### 5️⃣ Pipeline Execution

- Ran the pipeline in **Debug** mode to validate both activities
- Triggered the pipeline using **Trigger now** for a full (non-debug) execution
- Verified execution status and duration in the **Monitor** tab

📸 `screenshots/11_pipeline_run_succeeded.png`

---

### 6️⃣ IAM Roles

- Assigned the **Storage Blob Data Contributor** role to the ADF managed identity, scoped to the Storage Account
- Reviewed the **Reader** role at the Resource Group level for broader visibility

📸 `screenshots/12_iam_role_assignment.png`

---

### 🏁 Mini Project — End-to-End Validation

**Problem Statement:** Build a complete pipeline that reads a CSV file from Blob Storage and processes it using Azure Data Factory.

**Result:**
- ✅ Get Metadata activity successfully validated the source file (name, size, last modified)
- ✅ Copy Data activity successfully transferred the CSV from `source-data` to `destination-data`
- ✅ Pipeline executed successfully end-to-end (Debug + Trigger runs both Succeeded)
- ✅ IAM roles correctly scoped for secure, least-privilege access

📸 `screenshots/13_destination_container_final.png`

---

## 🐞 Issues & Fixes

| Issue | Cause | Fix |
|---|---|---|
| `RequestDisallowedByAzure` validation error while creating the Storage Account in **East US** | Azure for Students subscription restricts deployments to a specific set of "best available" regions | Switched the deployment region to **Central India**, which was supported by the subscription |
| "Field list" / dataset options not visible in Get Metadata activity config | UI panel needed a dataset selected first, or panel needed to be expanded/scrolled | Selected the dataset first, then expanded the bottom configuration panel |
| Pipeline "Validate" showed `parameter name is required` | Ran Validate before any activity was added to the canvas | Added the Get Metadata activity first, then re-validated (non-blocking, resolved automatically) |

---

## ✅ Key Learnings

- How to provision and organize Azure resources using **Resource Groups**
- Practical differences between **Linked Services**, **Datasets**, and **Activities** in ADF
- How **Get Metadata** can be used as a validation gate before triggering downstream activities like Copy Data
- How to debug and monitor pipeline runs using ADF Studio's **Debug** mode and **Monitor** tab
- How to apply **least-privilege access control** using Azure IAM / RBAC between services
- Troubleshooting real-world Azure subscription and region restrictions

---

## 🙋 Author

**Name:** Sumit kumar singh
**Internship:** Celebal Technologies — Data Engineering / Cloud Internship
**Week:** 4
**Date:** July 2026

---

## 📚 References

- [Azure Data Factory Documentation](https://learn.microsoft.com/en-us/azure/data-factory/)
- [Azure Blob Storage Documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/)
- [Azure RBAC Documentation](https://learn.microsoft.com/en-us/azure/role-based-access-control/overview)
