# Planning source manifest

The original uploaded PDFs are identified by exact-byte SHA-256 hashes in [manifest.json](manifest.json). Reviewable extracted text is committed here so source coverage can be checked without access to the original attachments:

- [System Specification v1.0](spec-v1.0.txt), sections 1-36.
- [Review Recommendations](review-recommendations.txt), pages 1-5.

The PDFs themselves are not committed. Text was extracted using `pdftotext -layout`; form-feed page separators were replaced with newlines. The extraction retains page numbers and headings but does not reproduce graphics or visual layout. Use the hash-matching original PDF to settle any extraction ambiguity. The manifest also hashes the exact committed UTF-8 text bytes.

Verify original PDF bytes with `sha256sum <original.pdf>` and compare `pdf_sha256`. Verify committed text using SHA-256 and compare `text_sha256`; file paths are relative to this directory. A source change requires a separate reviewed update to text and manifest, an explanation of the changed input, and an audit of §13 coverage, policies, and affected TODO tasks. Never overwrite sources silently or treat matching text as proof that a different PDF is identical.
