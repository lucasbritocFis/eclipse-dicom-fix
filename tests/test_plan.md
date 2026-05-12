# Test plan for eclipse-dicom-fix

## v0.1 acceptance criteria

A conversion is considered successful when:

1. **Output file is valid DICOM** — `pydicom.dcmread()` loads it without error
2. **Transfer syntax is correctly updated** — `file_meta.TransferSyntaxUID == "1.2.840.10008.1.2"`
3. **Pixel data is intact** — `numpy.array_equal(input_pixels, output_pixels)` for lossless inputs
4. **Original DICOM tags are preserved** — no patient info modification, no UID regeneration

## Test cases

### TC-01: JPEG Lossless single file
- Input: 1 CT slice in JPEG Lossless transfer syntax
- Expected: 1 output file in Implicit VR Little Endian, pixel array identical

### TC-02: JPEG Lossless full series
- Input: 200+ slices in JPEG Lossless
- Expected: all 200+ files converted, directory structure preserved, total time < 10s

### TC-03: Already-compatible files
- Input: file already in Implicit VR Little Endian
- Expected: file passed through unchanged (or copied byte-exact)

### TC-04: Non-DICOM file in directory
- Input: directory with mix of DICOM and non-DICOM (e.g., .txt, .DS_Store)
- Expected: non-DICOM files skipped silently or reported as skipped

### TC-05: Mixed transfer syntaxes
- Input: directory with multiple transfer syntaxes mixed
- Expected: each file converted appropriately based on its source syntax

### TC-06: Corrupt DICOM
- Input: file that pretends to be DICOM but is corrupted
- Expected: failure logged in report, other files in directory still processed

### TC-07: Eclipse import validation (manual)
- Input: TC-01 output
- Expected: imports cleanly into Eclipse TPS

## Status

- [ ] TC-01
- [ ] TC-02
- [ ] TC-03
- [ ] TC-04
- [ ] TC-05
- [ ] TC-06
- [ ] TC-07 (requires Eclipse access — manual validation)
