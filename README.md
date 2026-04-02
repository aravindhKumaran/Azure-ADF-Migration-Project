# Azure-ADF-Migration-Project

## Overview
This repository contains an **Azure Data Factory (ADF) pipeline** that migrates CSV files from an **on-premises file server** to **Azure Data Lake Storage Gen2 (ADLS)**. The pipeline is designed for **incremental processing**, ensuring that only new or updated files are migrated. Pipeline execution is tracked using a **JSON tracking file**, making it safe for reruns.

**Key Features:**
- Incremental load based on file creation timestamps
- Automated tracking of pipeline execution
- Self-hosted IR for secure on-prem access
- Parameterized datasets for dynamic file handling

---

## Architecture Diagram
![Architecture Diagram](images/architecture.png)

**Architecture Overview:**
On-Prem File Server --> (Self-Hosted IR) --> ADF Pipeline --> (Azure IR) --> ADLS Gen2


---

## Project Workflow

### Step 1: Initialize Pipeline
- Set pipeline variable `current_run_time` to `@utcNow()` at the start of execution.
- Lookup the tracking JSON in ADLS to get the `last_run_time`.

### Step 2: Retrieve Folder Metadata
- Use **Get Metadata activity** at folder level to list all files in the source folder.
- Output includes file names, types, and basic properties.

### Step 3: Process Each File
- **ForEach activity** iterates over all files in the folder.
- **File-level Get Metadata activity** retrieves `created` timestamp for each file.
- **If Condition** compares the file’s `created` timestamp with `last_run_time`:
  - **If newer:** Copy file to ADLS using the **Copy Activity**.
  - **If older:** Skip the file.

### Step 4: Copy Activity
- **Source:** On-premises CSV file.
- **Sink:** ADLS Gen2, partitioned and stored as CSV.
- Handles tabular translation and type conversion for seamless data migration.

### Step 5: Update Tracking JSON
- After processing all files, update the JSON with:
  - `last_run_time = current_run_time`
  - `last_run_status = SUCCESS`
  - `last_run_id = @pipeline().RunId`
  - `updated_at = @utcNow()`
- Ensures next pipeline run only processes new or updated files.

---

## Datasets

### Source Datasets
| Dataset | Purpose |
|---------|---------|
| `ds_OnPrem_csv_FolderLevel` | Retrieve folder-level metadata for all files |
| `ds_OnPrem_csv_dynamicFileName` | File-level metadata and copy activity; parameterized for dynamic file names |

### Sink Datasets
| Dataset | Purpose |
|---------|---------|
| `ds_adls_csv` | Stores migrated CSV files in ADLS |
| `ds_adls_template_tracking_json` | Template for pipeline tracking JSON |
| `ds_adls_json` | Stores updated pipeline execution info |

---

## Variables
| Name | Purpose |
|------|---------|
| `current_run_time` | Stores pipeline start time for tracking and incremental comparison |

---

## Getting Started

### Prerequisites
- Azure subscription with Data Factory and ADLS Gen2
- Self-hosted Integration Runtime configured for on-prem file access
- Access to the source folder containing CSV files

### Installation and Setup
1. **Clone the repository:**
```bash
git clone https://github.com/yourusername/DataMigration_OnPrem_To_ADLS.git
```
2. **Configure Linked Services:**
ls_OnPrem_FileServer for on-prem access via Self-Hosted IR
ls_ADLS for Azure Data Lake Storage Gen2

3. **Import Datasets:**
ds_OnPrem_csv_FolderLevel and ds_OnPrem_csv_dynamicFileName
Sink datasets: ds_adls_csv, ds_adls_template_tracking_json, ds_adls_json

4. **Import the ADF Pipeline:**
DataMigration_OnPrem_To_ADLS JSON
Publish to your Data Factory instance

5. **Verify Pipeline:**
Ensure the tracking JSON exists in ADLS
Run pipeline in debug mode for initial validation

### Notes
- Pipeline uses sequential ForEach for predictable incremental processing
- Incremental logic relies on tracking JSON for idempotency
- Ensure proper IR configuration and ADLS permissions before running
- Regular monitoring via ADF monitoring dashboard is recommended
- Git integration enables version control and collaborative updates
