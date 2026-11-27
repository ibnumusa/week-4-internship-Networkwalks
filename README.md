# week-4-internship-Networkwalks
# Mediroza General Hospital — Black-Box Penetration Test

## Executive Summary & Engagement Overview

This repository documents a Week 4 black-box penetration test against the simulated training target `medirozahospital.com` (`199.188.201.16`), conducted under written authorization from NetworkWalks. Across three technical milestones—**M1 (Initial Access)**, **M2 (Attack / Cracking)**, and **M3 (Data Extraction)**—authentication bypass via SQL injection, weak PDF password encryption, and an exposed SQL database backup were identified[cite: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 15, 16, 17].

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
| **Kali Linux (VirtualBox)** | Operating system used for active testing and command-line inspection |
| `whois` | Domain registration and name server lookup |
| `dnsrecon` | DNS record enumeration (`SOA`, `NS`, `MX`, `TXT`, `SPF`, `DMARC`, `SRV`) |
| `nslookup` | Domain-to-IP address resolution |
| `curl -I / -D` | Inspect HTTP response headers and raw server responses |
| `wafw00f` | Detect Web Application Firewall / edge proxy |
| `whatweb` | Fingerprint web server, CMS version, and metadata |
| `nikto` | Web server vulnerability and misconfiguration scanning |
| `gobuster` | Directory and file brute-forcing |
| `nmap` | Port and service version scanning (`-sV -Pn`) |
| **Browser DevTools + Manual SQLi** | Test and exploit login authentication logic |
| **NetworkWalks Hash Calculator** | Extract crackable `$pdf$...` hashes locally in-browser from encrypted PDFs |
| **NetworkWalks Password Cracker** | Dictionary attack against extracted `$pdf$...` hashes |
| `wget` / `cat` / Browser | Retrieve and inspect exposed `.sql` backup files[cite: 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13]

---

## 3. Findings & Proof of Exploitation

### 3.1 M1 — Initial Access (Authentication Bypass & Patient PDFs)

* **Reconnaissance & Fingerprinting:** Enumeration against `medirozahospital.com` resolved to `199.188.201.16` (`server274-1.web-hosting.com`), registered via NameCheap with `namecheaphosting.com` name servers and a privacy-protected registrant. DNS enumeration (`dnsrecon`) identified `SOA`, `NS`, `MX`, `SRV` (`_caldavs`, `_carddav`, `_autodiscover`), and `TXT` records, including a `p=none` DMARC policy. `wafw00f` and `whatweb` identified LiteSpeed and **Mediroza CMS 1.4.2**, while `nmap -sV -Pn` detected exposed services across FTP (`21/tcp` Pure-FTPd), SMTP (`25/tcp`, `26/tcp`, `465/tcp`, `587/tcp` Exim), DNS (`53/tcp` BIND), HTTP/HTTPS (`80/tcp`, `443/tcp` HAProxy/LiteSpeed), POP3 (`110/tcp`, `995/tcp` Dovecot), and IMAP (`143/tcp`, `993/tcp` Dovecot).
* **Directory & Header Misconfigurations:** `nikto` and `gobuster` flagged enabled directory indexing (`CWE-548`) on `/staff/`, `/patient/`, and `/old/`—all three also listed under `Disallow:` entries in `robots.txt`. `nikto` also noted missing security headers (`X-Content-Type-Options`, `Strict-Transport-Security`, `Content-Security-Policy`, `Referrer-Policy`, `Permissions-Policy`) and CMS version disclosure (`Mediroza CMS 1.4.2`).
* **SQL Injection Authentication Bypass:** Submitting `admin' --` in the Username field on `/patient/login.php` with an arbitrary password bypassed authentication and loaded `/patient/portal.php`. Submitting a bare single quote (`'`) triggered a raw MySQL syntax error (`mysqli_query(): You have an error in your SQL syntax...`), confirming unsanitized query concatenation.
* **Patient PDF Retrieval:** Inside *My lab reports* on `/patient/portal.php`, three password-protected lab reports were accessed and downloaded:
  * `patient_report_1.pdf` — *Pathology Report - S. Dlamini* (`Lab Ref LR-2024-1187`, `2024-11-04`)
  * `patient_report_2.pdf` — *Pathology Report - P. Reddy* (`Lab Ref LR-2024-1192`, `2024-11-05`)
  * `patient_report_3.pdf` — *Pathology Report - E. Thompson* (`Lab Ref LR-2024-1205`, `2024-11-06`)[cite: 1, 2, 3, 5, 6]

**Evidence:**
* `Screenshot 2026-11-26 182602.png` — SQL injection payload (`admin' --`) on `medirozahospital.com/patient/login.php`[cite: 1].
* `Screenshot 2026-11-26 182622.png` — Authenticated *My lab reports* portal (`medirozahospital.com/patient/portal.php`) listing all three encrypted PDFs[cite: 2].

---

### 3.2 M2 — Attack (Cracking Encrypted PDFs)

Each encrypted PDF (`Revision: R3`, `Version: V2`, `128-bit`) was parsed to extract a `$pdf$...` hash and cracked via dictionary wordlist attack[cite: 3, 4, 5, 6, 7, 8, 9, 10].

| PDF File | Patient / Report | Hash Extraction Tool | Wordlist Used | Recovered Password | Evidence Screenshot(s) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| `patient_report_1.pdf` | Sipho Dlamini (`MG-P-10231`, `LR-2024-1187`) | NetworkWalks Hash Calculator | Built-in list (100 entries) | `123456` | `Screenshot 2026-11-26 183128.png`, `Screenshot 2026-11-26 183910.png`, `image_8c1b1c.png`[cite: 3, 4, 22] |
| `patient_report_2.pdf` | Priya Reddy (`MG-P-10244`, `LR-2024-1192`) | NetworkWalks Hash Calculator | Built-in list (100 entries) | `password` | `Screenshot 2026-11-26 184113.png`, `Screenshot 2026-11-26 184221.png`, `Screenshot 2026-11-26 184228.png`, `image_8c1b3f.png`[cite: 5, 7, 8, 23] |
| `patient_report_3.pdf` | Emily Thompson (`MG-P-10258`, `LR-2024-1205`) | NetworkWalks Hash Calculator | `rockyou.txt` (73 words) | `!@#$%^&` | `Screenshot 2026-11-26 184210.png`, `Screenshot 2026-11-26 193611.png`, `Screenshot 2026-11-26 193654.png`, `image_8c1e43.png`[cite: 6, 9, 10, 24] |

Opening each unlocked PDF confirmed successful decryption and exposed confidential pathology laboratory results:
* **Sipho Dlamini (`MG-P-10231`, `LR-2024-1187`, DOB `1984-06-12`, Referring Dr: Dr. Anita Naicker, Whole blood EDTA):** Haemoglobin `13.9 g/dL` (Normal), White Cell Count `11.8 x10^9/L` (**HIGH**), Platelet Count `245 x10^9/L` (Normal), Fasting Glucose `5.4 mmol/L` (Normal), Creatinine `88 umol/L` (Normal)[cite: 22].
* **Priya Reddy (`MG-P-10244`, `LR-2024-1192`, DOB `1991-02-28`, Referring Dr: Dr. Johan van der Merwe, Serum):** Total Cholesterol `6.4 mmol/L` (**HIGH**), LDL Cholesterol `4.1 mmol/L` (**HIGH**), HDL Cholesterol `1.2 mmol/L` (Normal), Triglycerides `1.8 mmol/L` (**HIGH**), Fasting Glucose `4.9 mmol/L` (Normal)[cite: 23].
* **Emily Thompson (`MG-P-10258`, `LR-2024-1205`, DOB `1978-09-03`, Referring Dr: Dr. Ahmed Kara, Serum):** Haemoglobin `11.4 g/dL` (**LOW**), Ferritin `9 ug/L` (**LOW**), TSH `3.1 mIU/L` (Normal), Vitamin D `42 nmol/L` (**LOW**), Fasting Glucose `5.1 mmol/L` (Normal)[cite: 24].

---

### 3.3 M3 — Data Extraction (Staff Salaries & Shareholder Records)

Browsing `https://medirozahospital.com/old/` revealed an open LiteSpeed directory index (`Index of /old/`) containing `mediroza_db_backup_2019.sql` (`7k`, last modified `2026-09-04 02:02`)[cite: 13]. Reviewing the SQL dump (`mediroza_hr` database) exposed two sensitive tables:

* **`staff` (30 records):** Contains `id`, `full_name`, `job_title`, `department`, `email`, `phone`, `national_id`, `monthly_salary_zar`, and `date_joined` across hospital roles (from `Medical Director` and `Chief Financial Officer` down to nurses, technicians, and ward clerks).
* **`shareholders` (10 records):** Contains `id`, `shareholder_name`, `share_percent`, `shares_held`, and `share_class` (`Ordinary` and `Preferential`), showing ownership stakes held by hospital staff/doctors (such as Dr. Rajesh Naidoo, Dr. Johan van der Merwe, Thabo Molefe, Sarah Botha, Dr. Ahmed Kara, Naledi Zulu, Michael Roberts, and Dr. Vikram Chetty) as well as external entities (`Cedar Health Holdings (Pty) Ltd` and `Reddy Family Trust`)[cite: 11, 12].

**Evidence:**
* `Screenshot 2026-11-27 142849.png` — Open directory listing at `medirozahospital.com/old/` exposing `mediroza_db_backup_2019.sql`[cite: 13].
* `Screenshot 2026-11-26 185922.png` — Backup header, `staff` schema, and initial employee records (`1`–`20`)[cite: 11].
* `Screenshot 2026-11-26 185931.png` — Remaining `staff` records (`12`–`30`), `shareholders` schema, and all `10` shareholder records[cite: 12].

---

## 4. Risk Rating & Impact

| # | Risk / Finding | Evidence / Observation | Potential Impact | Risk Level |
| :--- | :--- | :--- | :--- | :--- |
| 1 | **SQL injection authentication bypass** | `admin' --` bypasses login on `/patient/login.php`; bare `'` triggers raw `mysqli_query()` SQL syntax error[cite: 1, 2] | Full unauthorized access to patient accounts/records and potential portal compromise | ● **Critical** |
| 2 | **Sensitive database backup publicly downloadable** | `mediroza_db_backup_2019.sql` exposed in `/old/` with full `staff` (30 rows) and `shareholders` (10 rows) tables[cite: 11, 12, 13] | Mass exposure of employee salaries (`monthly_salary_zar`), national IDs, contact details, and corporate ownership | ● **Critical** |
| 3 | **Directory indexing enabled** | Open index listing on `/old/` (plus `/staff/` and `/patient/` flagged in scan)[cite: 13] | Allows direct browsing and enumeration of exposed files without guessing filenames | ● **Critical** |
| 4 | **`robots.txt` discloses sensitive paths** | `Disallow: /patient/`, `/staff/`, `/old/` explicitly lists restricted/internal directories | Directs attackers to high-value targets while providing zero access enforcement | ● **High** |
| 5 | **Confidential patient PDFs retrievable and weakly encrypted** | Three encrypted patient PDFs cracked via short/common wordlist entries (`123456`, `password`, `!@#$%^&`) and opened to expose lab results[cite: 2, 4, 8, 10, 22, 23, 24] | Breach of patient medical confidentiality; weak passwords defeat PDF encryption | ● **High** |
| 6 | **Missing security headers** | Missing `X-Content-Type-Options`, `Strict-Transport-Security`, `Content-Security-Policy`, `Referrer-Policy`, `Permissions-Policy` | Increased exposure to clickjacking, MIME-type sniffing, downgrade, and injection risks | ● **Medium** |
| 7 | **Server / CMS version disclosure** | `Mediroza CMS 1.4.2` and LiteSpeed server banners exposed | Aids targeted vulnerability research against specific software versions | ● **Low** |

**Risk Level Key:** ● **Critical** | ● **High** | ● **Medium** | ● **Low**

---

## 5. Recommendations & Remediation

1. **Remediate SQL Injection Immediately:** Replace all string-concatenated SQL queries with parameterized prepared statements and strict input validation across all authentication and data endpoints.
2. **Remove Exposed Backups from Web Root:** Delete `mediroza_db_backup_2019.sql` from `/old/` immediately and store database backups offline or on restricted internal storage outside the public document root.
3. **Disable Directory Indexing & Enforce Server Access Controls:** Disable directory auto-indexing (`Options -Indexes` or LiteSpeed equivalent) across all directories and enforce proper authentication/authorization checks on `/staff/` and `/patient/`.
4. **Remove Sensitive Paths from `robots.txt`:** Do not rely on `robots.txt` to conceal sensitive directories; remove `Disallow` hints for internal/restricted paths and protect them with server-side controls.
5. **Enforce Strong Document Passwords / Encryption Policy:** Require strong, high-entropy passwords or secure portal-authenticated viewing rather than predictable or dictionary-crackable passwords for confidential patient files.
6. **Configure Security HTTP Headers:** Add `Strict-Transport-Security`, `X-Content-Type-Options: nosniff`, `Content-Security-Policy`, `Referrer-Policy`, and `Permissions-Policy` at the web server or application layer.
7. **Suppress Verbose Database Errors:** Disable verbose SQL/database error output in production environments and return generic error responses without leaking query structure or backend details.
8. **Harden Banner Exposure & Patch CMS:** Minimize version disclosure where configurable and keep the CMS and server stack patched and up to date.

---

## 6. Evidence Collected (Screenshots)

### M1 — Initial Access
* `Screenshot 2026-11-26 182602.png` — SQL injection payload (`admin' --`) submitted in the Username field on `medirozahospital.com/patient/login.php`[cite: 1].
* `Screenshot 2026-11-26 182622.png` — Authenticated Patient Portal view (`medirozahospital.com/patient/portal.php`) under *My lab reports*, showing downloadable encrypted PDFs for **S. Dlamini**, **P. Reddy**, and **E. Thompson**[cite: 2].

### M2 — Attack (Cracking)
* `Screenshot 2026-11-26 183128.png` — Extracted `$pdf$...` hash for `patient_report_1.pdf` (`3.6 KB`, `Revision: R3`, `Version: V2`, `128 bit`) in NetworkWalks Hash Calculator[cite: 3].
* `Screenshot 2026-11-26 183910.png` — NetworkWalks Password Cracker matching password `123456` against the `patient_report_1.pdf` hash using the 100-entry built-in wordlist[cite: 4].
* `image_8c1b1c.png` — Decrypted Pathology Laboratory Report for **Sipho Dlamini** (`MG-P-10231`, `LR-2024-1187`)[cite: 22].
* `Screenshot 2026-11-26 184113.png` — Extracted `$pdf$...` hash for `patient_report_2.pdf` (`3.5 KB`, `Revision: R3`, `Version: V2`, `128 bit`) in NetworkWalks Hash Calculator[cite: 5].
* `Screenshot 2026-11-26 184221.png` — Dictionary attack in progress on `patient_report_2.pdf` hash (`Trying: 123456`, 100-word built-in list)[cite: 7].
* `Screenshot 2026-11-26 184228.png` — Successful crack of `patient_report_2.pdf` hash revealing password `password`[cite: 8].
* `image_8c1b3f.png` — Decrypted Pathology Laboratory Report for **Priya Reddy** (`MG-P-10244`, `LR-2024-1192`)[cite: 23].
* `Screenshot 2026-11-26 184210.png` — Extracted `$pdf$...` hash for `patient_report_3.pdf` (`3.6 KB`, `Revision: R3`, `Version: V2`, `128 bit`) in NetworkWalks Hash Calculator[cite: 6].
* `Screenshot 2026-11-26 193611.png` — `patient_report_3.pdf` hash loaded into NetworkWalks Password Cracker with `rockyou.txt` (73 words) active[cite: 9].
* `Screenshot 2026-11-26 193654.png` — Successful crack of `patient_report_3.pdf` hash revealing password `!@#$%^&`[cite: 10].
* `image_8c1e43.png` — Decrypted Pathology Laboratory Report for **Emily Thompson** (`MG-P-10258`, `LR-2024-1205`)[cite: 24].

### M3 — Data Extraction
* `Screenshot 2026-11-27 142849.png` — Open LiteSpeed directory index at `medirozahospital.com/old/` displaying `mediroza_db_backup_2019.sql` (`2026-09-04 02:02`, `7k`)[cite: 13].
* `Screenshot 2026-11-26 185922.png` — Contents of `mediroza_db_backup_2019.sql` showing the `med_backup` header (`mediroza_hr`), `staff` table schema, and employee records (`1` through `20`)[cite: 11].
* `Screenshot 2026-11-26 185931.png` — Remaining `staff` records (`12` through `30`) plus the `shareholders` table schema and all `10` shareholder records[cite: 12].
