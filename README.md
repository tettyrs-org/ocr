# OCR — system overview

Index of the public repositories that make up the OCR document system. Each component lives in its own
repository. This repository contains only this overview.

| Repository | Visibility | What it does | Stack |
|---|---|---|---|
| [ocr-app](https://github.com/tettyrs-org/ocr-app) | Public | Web and mobile app to upload, track, and review documents | Flutter, Firebase Hosting, Cloud Functions (BFF) |
| [ocr-api](https://github.com/tettyrs-org/ocr-api) | Public | REST API: document lifecycle, storage, verification, audit | Java, Quarkus, PostgreSQL, MinIO |
| [ms-ocr](https://github.com/tettyrs-org/ms-ocr) | Public | Extraction and correction of OCR text into document fields | Java, Quarkus, Redis streams |
| [ocr-engine](https://github.com/tettyrs-org/ocr-engine) | Public | OCR service for images and PDFs: text, coordinates, KTP NIK mode, optional vision companion | Python, FastAPI, Tesseract, Qwen2.5-VL (Ollama) |

## How they fit together

```
ocr-app  --HTTPS-->  BFF (Cloud Functions)  --HTTPS-->  ocr-api
                                                          |
                                      queue (Redis) ------+------ storage (MinIO)
                                                          |
                                                          v
                                                       ms-ocr  --HTTP-->  ocr-engine
                                                                             |
                                                                             +--> Ollama (optional vision)
```

1. The app uploads a file through the BFF to `ocr-api`.
2. `ocr-api` stores the file and queues an extraction job.
3. `ms-ocr` takes the job, asks `ocr-engine` for the text, and extracts the fields.
4. `ocr-engine` runs Tesseract. For identity cards it also reads the NIK, and for missing fields it
   can ask a local vision model.
5. Results come back to `ocr-api`. A reviewer checks and saves them in the app.

## Status

Work in progress. Accuracy has been measured on synthetic fixtures and a small number of real
samples only. Do not use the system for production decisions before a full evaluation.

## License

Not yet determined.
