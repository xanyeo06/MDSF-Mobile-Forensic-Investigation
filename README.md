# MDSF — Mobile Forensic Investigation

A simulated corporate data-theft investigation in which an employee was suspected of using a disguised
"Calculator" vault application to photograph confidential financial documents on a personal Android device.
Evidence was recovered through ADB logical extraction, preserved as a hash-verified AD1 image, and analysed in
Autopsy and ExifTool, with APK static analysis explaining how the application concealed its contents.

## Contents

| File | Description |
| --- | --- |
| `MDSF-Assignment-Report.docx` | Full investigation report |

## Scenario

On 4 July 2026 corporate security was alerted to an insider threat involving the unauthorised duplication of
sensitive financial records. A whistleblower reported seeing the suspect photograph a company monitor with a
personal smartphone, then spend an unusual amount of time in what appeared to be a "Calculator" application.
A preliminary manual inspection of the Gallery and accessible file system found nothing, so investigators
suspected a disguised vault app — software that presents a working calculator while hiding a PIN-protected
storage area. The objective was to logically extract the device via ADB, locate the vault's obfuscated storage
directory, and confirm the nature and origin of the recovered files.

## Methodology

1. **Evidence acquisition** — `adb devices`, then `adb shell pm list packages` to identify the suspicious
   package `com.calculator.hideu`, `ls -la` on `/sdcard/` to surface the hidden `.dont_delete_me_by_hideu`
   directory, and `adb pull` to extract it.
2. **APK static analysis** — the APK was located with `adb shell pm path`, pulled, and decompiled with APKTool
   to inspect the decoded `AndroidManifest.xml`.
3. **Evidence preservation** — FTK Imager created an AD1 logical image of the extracted folder; matching MD5
   and SHA1 hashes confirmed the copy was unaltered.
4. **Analysis setup** — Autopsy does not read AD1 natively, so the image was mounted read-only as a virtual
   drive in FTK Imager and added to Autopsy via Logical Files, with the File Type Identification and Extension
   Mismatch Detector ingest modules enabled.
5. **Evidence analysis and recovery** — extension-mismatch results, rendering of the recovered files, and hex
   signature inspection.
6. **File obfuscation analysis** — documenting the scrambling, extension stripping and timestamp findings.
7. **EXIF metadata analysis** — content-level examination in ExifTool, independent of the filesystem.

## Tools used

| Tool | Purpose |
| --- | --- |
| ADB | Device connection, package enumeration, hidden directory discovery and logical extraction |
| FTK Imager | Hash-verified AD1 logical imaging and read-only image mounting |
| Autopsy | Case management, content-based file type identification, extension mismatch detection |
| ExifTool | Content-level metadata extraction, unaffected by filesystem timestamp stripping |
| APKTool 3.0.2 | Decompilation of the APK to inspect the manifest and declared permissions |

## Findings

- The vault stored files in shared storage, so no root access was required — a design trade-off that made the
  evidence readable by a standard ADB logical extraction.
- Filenames were scrambled into random alphanumeric strings and extensions stripped entirely; Autopsy still
  identified every file as `image/jpeg` from content alone, and the `FF D8 FF E0` magic bytes proved the format
  regardless of filename.
- All three recovered files rendered as photographs of a monitor displaying a confidential document, with
  Modified Times between 10:57:16 and 10:57:20 SGT on 4 July 2026, matching the incident timeline.
- The manifest declared permissions far beyond a calculator's needs — CAMERA, RECORD_AUDIO, READ_CONTACTS,
  INTERNET and MANAGE_EXTERNAL_STORAGE — so concealed material could extend beyond still images.
- It also declared a `DriveBackupService` component and a `GoogleLoginHelper` activity, indicating undisclosed
  cloud backup, while `android:allowBackup="false"` deliberately kept vault contents out of native backups.
- ExifTool corroborated the JPEG finding and dated the file to 4 July 2026; the Display P3 ICC profile
  attributed to Google Inc. 2016 is characteristic of the Android camera pipeline, and the 1132x1280
  resolution suits a photo of a screen. No GPS, device make/model or original capture time remained, showing
  the app systematically sanitises metadata on ingestion.
- Together the evidence confirmed the files were JPEG photographs captured on an Android device and stored in
  the vault that day, supporting the whistleblower's account.

## Evidence set

The investigation worked from three recovered photographs, which show a "Highly Confidential" M&A planning
document displayed on a monitor, plus a set of distractor files kept alongside them: client notes, an employee
salary spreadsheet, an internal network diagram, an internal report, a meeting agenda, project notes, a holiday
photo, a payslip and a resume. The images and the screenshots of the tooling are reproduced in the report.

## Recommendations

- Deploy MDM with application whitelisting to block unauthorised apps, including disguised vaults, on
  corporate-issued devices.
- Use MDM geofencing to disable device cameras on the corporate network and in sensitive physical zones.
- Enforce DLP and clean-desk policies governing how confidential documents are displayed and how personal
  devices may be used where such data is handled.
- Extend the investigation to cloud storage by examining the suspect's linked Google account activity around
  the time of the incident.

## Key takeaways

- Logical extraction can defeat a vault app when it writes to shared storage instead of app-private storage.
- Filenames and extensions are cosmetic; file signatures and content-based identification are authoritative.
- Metadata is layered — ExifTool's content-level view resolved what zeroed filesystem timestamps could not.
- Static APK analysis turned a recovered-files case into a statement about deliberate design intent.
- Tool incompatibility (Autopsy and AD1) is a workflow problem to solve without breaking chain of custody.

## Notes

- This is a simulated exercise conducted on a lab Android device; the "Contoso" documents are fabricated and no
  real personal data is included.
- The investigation was limited to a logical extraction: unallocated space, deleted database entries and
  app-private storage were out of scope, and cloud synchronisation could not be verified without account access.
