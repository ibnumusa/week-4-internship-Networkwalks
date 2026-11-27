# week-4-internship-Networkwalks
# Mediroza General Hospital — Black-Box Penetration Test

## Executive Summary & Engagement Overview

This report documents a Week 4 black-box penetration test against the simulated training target `medirozahospital.com` (`199.188.201.16`), conducted under written authorization from NetworkWalks. Across three technical milestones—**M1 (Initial Access)**, **M2 (Attack / Cracking)**, and **M3 (Data Extraction)**—authentication bypass via SQL injection, weak PDF password encryption, and an exposed SQL database backup were identified[cite: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 15, 16, 17].

| Field | Details |
| :--- | :--- |
| **Modules Completed** | M1 — Initial Access, M2 — Attack (Cracking), M3 — Data Extraction |
| **Client / Target** | Mediroza General Hospital — `https://medirozahospital.com` (`199.188.201.16`) |
| **Authorization Status** | Yes — Written authorization granted by NetworkWalks for this simulated training target |
| **Engagement Type** | Full Black-Box Penetration Test[cite: 1, 15, 16, 17] |

---

## 1. Liability Disclaimer

I performed these activities only against the target explicitly assigned and authorised by NetworkWalks for this capstone project. All materials in this report and repository are for educational and assessment purposes only. Do not use anything here to break the law. The instructor, the authors, and NetworkWalks are not responsible for what you do with this knowledge. Every action you take is your own responsibility. Misuse of these techniques against systems you do not own or have no permission to test is a crime in most countries, even when nothing is damaged[cite: 15, 16, 17, 18, 19, 20, 21].

---

## 2. Scope, Rules & Methodology

### Scope & Rules of Engagement
Testing was strictly limited to `https://medirozahospital.com` as a black-box assessment under NetworkWalks authorization, with no social engineering, denial-of-service testing, or out-of-scope actions performed[cite: 1, 15, 16].

### Tools Used

| Tool / Environment | Purpose |
| :--- | :--- |
| `whois` | Domain registration, creation/expiry dates, and name server lookup |
| `nmap` | Port and service version scanning |
| `curl` (`curl -i -X POST -d "username=admin' -- &password=test"`, etc.) | Send HTTP/POST requests and test login authentication logic |
| **Manual SQL Injection (including true statements)** | Test and bypass login authentication logic |
| **NetworkWalks Hash Calculator** | Extract crackable `$pdf$...` hashes locally in-browser from encrypted PDFs |
| **NetworkWalks Password Cracker** | Dictionary attack against extracted `$pdf$...` hashes |
| **Browser / File Viewer** | Access the portal, view exposed directory/SQL backup data, and open decrypted PDFs[cite: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13]

---

## 3. Findings & Proof of Exploitation

### 3.1 M1 — Initial Access (Authentication Bypass & Patient PDFs)

* **Reconnaissance & Fingerprinting:** Using `whois`, `nmap`, and `curl`, the target domain `medirozahospital.com` (`199.188.201.16`) and its exposed web/portal services were inspected.
* **SQL Injection Authentication Bypass:** Testing `/patient/login.php` with SQL injection payloads—including comment-based bypass (`admin' --`) and true-condition statements—bypassed authentication and returned the authenticated portal (`medirozahospital.com/patient/portal.php`), verified both in the browser and via `curl -i -X POST -d "username=admin' -- &password=test"`[cite: 1, 2].
* **Patient PDF Retrieval:** Inside *My lab reports* on `/patient/portal.php`, three password-protected lab reports were accessed and downloaded:
  * `patient_report_1.pdf` — *Pathology Report - S. Dlamini* (`Lab Ref LR-2024-1187`, `2024-11-04`)
  * `patient_report_2.pdf` — *Pathology Report - P. Reddy* (`Lab Ref LR-2024-1192`, `2024-11-05`)
  * `patient_report_3.pdf` — *Pathology Report - E. Thompson* (`Lab Ref LR-2024-1205`, `2024-11-06`)[cite: 2, 3, 5, 6]

---

### 3.2 M2 — Attack (Cracking Encrypted PDFs)

Each encrypted PDF (`Revision: R3`, `Version: V2`, `128-bit`) was parsed to extract a `$pdf$...` hash and cracked via dictionary wordlist attack[cite: 3, 4, 5, 6, 7, 8, 9, 10].

| PDF File | Patient / Report | Hash Extraction Tool | Wordlist Used | Recovered Password |
| :--- | :--- | :--- | :--- | :--- |
| `patient_report_1.pdf` | Sipho Dlamini (`MG-P-10231`, `LR-2024-1187`) | NetworkWalks Hash Calculator | Built-in list (100 entries) | `123456` |
| `patient_report_2.pdf` | Priya Reddy (`MG-P-10244`, `LR-2024-1192`) | NetworkWalks Hash Calculator | Built-in list (100 entries) | `password` |
| `patient_report_3.pdf` | Emily Thompson (`MG-P-10258`, `LR-2024-1205`) | NetworkWalks Hash Calculator | `rockyou.txt` (73 words) | `!@#$%^&`[cite: 3, 4, 5, 6, 7, 8, 9, 10, 22, 23, 24]

Opening each unlocked PDF confirmed successful decryption and exposed confidential pathology laboratory results:
* **Sipho Dlamini (`MG-P-10231`, `LR-2024-1187`, DOB `1984-06-12`, Referring Dr: Dr. Anita Naicker, Whole blood EDTA):** Haemoglobin `13.9 g/dL` (Normal), White Cell Count `11.8 x10^9/L` (**HIGH**), Platelet Count `245 x10^9/L` (Normal), Fasting Glucose `5.4 mmol/L` (Normal), Creatinine `88 umol/L` (Normal)[cite: 22].
* **Priya Reddy (`MG-P-10244`, `LR-2024-1192`, DOB `1991-02-28`, Referring Dr: Dr. Johan van der Merwe, Serum):** Total Cholesterol `6.4 mmol/L` (**HIGH**), LDL Cholesterol `4.1 mmol/L` (**HIGH**), HDL Cholesterol `1.2 mmol/L` (Normal), Triglycerides `1.8 mmol/L` (**HIGH**), Fasting Glucose `4.9 mmol/L` (Normal)[cite: 23].
* **Emily Thompson (`MG-P-10258`, `LR-2024-1205`, DOB `1978-09-03`, Referring Dr: Dr. Ahmed Kara, Serum):** Haemoglobin `11.4 g/dL` (**LOW**), Ferritin `9 ug/L` (**LOW**), TSH `3.1 mIU/L` (Normal), Vitamin D `42 nmol/L` (**LOW**), Fasting Glucose `5.1 mmol/L` (Normal)[cite: 24].

---

### 3.3 M3 — Data Extraction (Staff Salaries & Shareholder Records)

Browsing `https://medirozahospital.com/old/` revealed an open LiteSpeed directory index (`Index of /old/`) containing `mediroza_db_backup_2019.sql` (`7k`, last modified `2026-09-04 02:02`)[cite: 13]. Reviewing the SQL dump (`mediroza_hr` database) exposed two sensitive tables:

* **`staff` (30 records):** Contains `id`, `full_name`, `job_title`, `department`, `email`, `phone`, `national_id`, `monthly_salary_zar`, and `date_joined` across hospital roles (from `Medical Director` and `Chief Financial Officer` down to nurses, technicians, and ward clerks).
* **`shareholders` (10 records):** Contains `id`, `shareholder_name`, `share_percent`, `shares_held`, and `share_class` (`Ordinary` and `Preferential`), showing ownership stakes held by hospital staff/doctors (such as Dr. Rajesh Naidoo, Dr. Johan van der Merwe, Thabo Molefe, Sarah Botha, Dr. Ahmed Kara, Naledi Zulu, Michael Roberts, and Dr. Vikram Chetty) as well as external entities (`Cedar Health Holdings (Pty) Ltd` and `Reddy Family Trust`)[cite: 11, 12].

---

## 4. Risk Rating & Impact

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **SQL injection authentication bypass** | Comment (`admin' --`) and true-condition payloads bypass login on `/patient/login.php` (confirmed via browser and `curl -i -X POST -d "username=admin' -- &password=test"`) | Full unauthorized access to patient accounts/records and potential portal compromise | ● **Critical** |
| 2 | **Sensitive database backup publicly downloadable** | `mediroza_db_backup_2019.sql` exposed in `/old/` with full `staff` (30 rows) and `shareholders` (10 rows) tables | Mass exposure of employee salaries (`monthly_salary_zar`), national IDs, contact details, and corporate ownership | ● **Critical** |
| 3 | **Directory indexing enabled** | Open directory listing (`Index of /old/`) exposing backup files directly in the browser | Allows direct browsing and enumeration of exposed files without guessing filenames | ● **Critical** |
| 4 | **Confidential patient PDFs retrievable and weakly encrypted** | Three encrypted patient PDFs cracked via short/common wordlist entries (`123456`, `password`, `!@#$%^&`) and opened to expose lab results | Breach of patient medical confidentiality; weak passwords defeat PDF encryption[cite: 1, 2, 4, 8, 10, 11, 12, 13, 22, 23, 24] | ● **High** |

**Risk Level Key:** ● **Critical** | ● **High** | ● **Medium** | ● **Low**

---

## 5. Recommendations & Remediation

1. **Remediate SQL Injection Immediately:** Replace all string-concatenated SQL queries with parameterized prepared statements and strict input validation across all authentication and data endpoints.
2. **Remove Exposed Backups from Web Root:** Delete `mediroza_db_backup_2019.sql` from `/old/` immediately and store database backups offline or on restricted internal storage outside the public document root.
3. **Disable Directory Indexing & Enforce Server Access Controls:** Disable directory auto-indexing (`Options -Indexes` or LiteSpeed equivalent) across all directories and enforce proper authentication/authorization checks on restricted paths.
4. **Enforce Strong Document Passwords / Encryption Policy:** Require strong, high-entropy passwords or secure portal-authenticated viewing rather than predictable or dictionary-crackable passwords for confidential patient files.2849.png` — Open LiteSpeed directory index at `medirozahospital.com/old/` displaying `mediroza_db_backup_2019.sql` (`2026-09-04 02:02`, `7k`)[cite: 13].
* `Screenshot 2026-11-26 185922.png` — Contents of `mediroza_db_backup_2019.sql` showing the `med_backup` header (`mediroza_hr`), `staff` table schema, and employee records (`1` through `20`)[cite: 11].
* `Screenshot 2026-11-26 185931.png` — Remaining `staff` records (`12` through `30`) plus the `shareholders` table schema and all `10` shareholder records[cite: 12].
