# eclipse-dicom-fix

> Re-encode DICOM files in compressed transfer syntaxes to **Implicit VR Little Endian** for clean import into Varian Eclipse Treatment Planning System.

**Status:** 🚧 v0.1 in development

---

## The clinical problem this solves

Varian Eclipse TPS imports DICOMs reliably only in uncompressed transfer syntaxes — primarily **Implicit VR Little Endian** (`1.2.840.10008.1.2`) and Explicit VR Little Endian. When DICOM data arrives in **JPEG Lossless** (`1.2.840.10008.1.2.4.70`), JPEG 2000, or similar compressed syntaxes, Eclipse refuses the import with cryptic errors.

This happens routinely in clinical practice:

- Patients arrive with external CT scans on CD from imaging centers that compress DICOM to save storage
- PET-CT or MR studies come from PACS systems that auto-compress on archive
- Third-party segmentation tools (TotalSegmentator, AI vendors) output compressed DICOM
- Legacy archives store everything compressed

The current workaround is manual: open every file in OsiriX/Horos and re-export, or run `gdcmconv` per file. Both are slow and error-prone for full CT series with hundreds of slices.

This tool runs in seconds on a directory and produces an Eclipse-ready copy with original structure preserved.

---

## What it does

- Walks a source directory recursively
- Detects DICOM files in compressed transfer syntaxes
- Decompresses pixel data
- Re-encodes to Implicit VR Little Endian
- Mirrors the directory structure in the output folder
- Skips files already in Eclipse-compatible syntaxes
- Reports converted / skipped / failed counts

---

## Supported input transfer syntaxes

| UID | Name | Status |
|-----|------|--------|
| `1.2.840.10008.1.2.4.70` | JPEG Lossless | ✅ v0.1 |
| `1.2.840.10008.1.2.4.80` | JPEG-LS Lossless | ✅ v0.1 |
| `1.2.840.10008.1.2.4.81` | JPEG-LS Near-Lossless | ✅ v0.1 |
| `1.2.840.10008.1.2.4.90` | JPEG 2000 Lossless | 🚧 v0.2 |
| `1.2.840.10008.1.2.4.91` | JPEG 2000 | 🚧 v0.2 |
| `1.2.840.10008.1.2.5`    | RLE Lossless | 🚧 v0.2 |

Files already in Implicit/Explicit VR Little Endian pass through unchanged.

---

## Installation

```bash
git clone https://github.com/lucasbritocFis/eclipse-dicom-fix.git
cd eclipse-dicom-fix
pip install -r requirements.txt
```

---

## Usage

```bash
# Directory of compressed DICOMs → directory of Eclipse-ready DICOMs
python eclipse_dicom_fix.py /path/to/incoming-ct/ /path/to/eclipse-ready/

# Single file
python eclipse_dicom_fix.py /path/to/file.dcm /path/to/output.dcm

# Verbose mode (per-file logging)
python eclipse_dicom_fix.py /path/to/incoming-ct/ /path/to/eclipse-ready/ -v
```

---

## Example output

```
2026-05-12 09:14:22  INFO  Found 247 DICOM files in /home/lucas/incoming-ct/
2026-05-12 09:14:25  INFO  Converted 247 files in 3.1s
Converted: 247 | Skipped (already compatible): 0 | Failed: 0
```

---

## How it works

```
Input DICOM
    ↓
pydicom.dcmread()
    ↓
Inspect TransferSyntaxUID
    ↓
[If compressed]
    ds.decompress()  ← uses pylibjpeg backend
    ↓
ds.file_meta.TransferSyntaxUID = ImplicitVRLittleEndian
    ↓
ds.save_as(output_path)
    ↓
Output DICOM (Eclipse-ready)
```

---

## Roadmap

- [x] v0.1 scaffold
- [ ] v0.1 — JPEG Lossless / JPEG-LS support, directory mode, basic CLI
- [ ] v0.2 — JPEG 2000, RLE Lossless
- [ ] v0.3 — Pre/post-conversion validation (CT pixel array sanity check)
- [ ] v0.4 — Streamlit web UI for non-developer physicists
- [ ] v0.5 — Tests against multiple TPS (RayStation, Monaco) to expand "TPS-friendly" output

---

## Caveats

- Only handles transfer syntax conversion. Does **not** modify any other DICOM tags — no de-identification, no UID regeneration.
- Lossless input produces bit-exact lossless output. Lossy compressed input remains lossy after decompression.
- This tool is **not a medical device** and is **not validated for clinical decision making**. Always verify the converted DICOM imports correctly into your TPS before clinical use.
- Tested against real-world Siemens, GE, and Philips CT exports. Open an issue if your scanner produces something this tool can't handle.

---

## Why this exists

Built by a clinical medical physicist who got tired of re-exporting external CT scans manually. Open-sourced because every radiotherapy clinic in the world has hit this problem at least once.

See also: [dicom-rt-explained](https://github.com/lucasbritocFis/dicom-rt-explained) for a broader introduction to DICOM-RT.

---

## License

MIT — see [LICENSE](./LICENSE).

## Author

**Lucas Brito, PhD** — Clinical Medical Physicist and Software Developer.
[GitHub](https://github.com/lucasbritocFis) · Rio de Janeiro, Brazil
