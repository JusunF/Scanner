# Scanner

A self-hosted document scanner web app. Upload a PDF, Word document, text file, or image; Scanner extracts the text (using OCR when needed), stores it, and lets you view it, convert it to other formats, or delete it later. It also includes a small tool that converts a video link to an MP3.

Built with FastAPI, PostgreSQL, and Tesseract OCR.

## Features

- **Upload & scan**: supports `.pdf`, `.docx`, `.txt`, `.png`, `.jpg`, `.jpeg`, `.tiff`, `.tif`, `.bmp`, `.webp`
- **Smart text extraction**: reads embedded text from PDFs and DOCX files; falls back to OCR for scanned pages and images
- **Past scans**: every upload is saved to PostgreSQL so you can come back and view the extracted text
- **Convert**: export the extracted text as `pdf`, `docx`, `txt`, `png`, or `jpg` (image uploads converted to PDF keep their original look)
- **Delete**: remove a document and its stored file
- **Link to MP3**: paste a video link and download the audio as an MP3 (via yt-dlp and ffmpeg)

## Tech stack

| Area | Tools |
| --- | --- |
| Web framework | FastAPI, Uvicorn |
| Database | PostgreSQL, SQLAlchemy, psycopg |
| OCR | Tesseract, pytesseract, Pillow |
| PDF / DOCX reading | PyMuPDF, python-docx |
| Conversion | reportlab, python-docx, Pillow |
| Audio | yt-dlp, ffmpeg |
| Frontend | Plain HTML + JavaScript (`static/index.html`) |

## Project structure

```
Scanner/
├── README.md
└── scanner/
    ├── requirements.txt
    ├── .env                  # your local settings (not committed)
    ├── static/
    │   └── index.html        # web interface
    └── app/
        ├── main.py           # API routes
        ├── database.py       # DB engine and session
        ├── models.py         # Document table
        └── services/
            ├── extractor.py  # text extraction + OCR
            └── converter.py  # format conversion
```

## Prerequisites

- **Python 3.10+**
- **PostgreSQL** with a database created for this app
- **Tesseract OCR**: on Windows, install from the [UB Mannheim builds](https://github.com/UB-Mannheim/tesseract/wiki)
- **ffmpeg** (only needed for the link-to-MP3 feature): on Windows, `winget install Gyan.FFmpeg`

## Setup

1. **Clone the repo**
   ```
   git clone https://github.com/JusunF/Scanner.git
   cd Scanner/scanner
   ```

2. **Create and activate a virtual environment**
   ```
   python -m venv venv
   venv\Scripts\activate
   ```

3. **Install dependencies**
   ```
   pip install -r requirements.txt
   ```

4. **Create a `.env` file** in the `scanner/` folder:
   ```
   DATABASE_URL=postgresql+psycopg://USER:PASSWORD@localhost:6699/DBNAME
   TESSERACT_CMD=C:\Program Files\Tesseract-OCR\tesseract.exe
   OCR_LANG=eng
   ```
   - `DATABASE_URL` is required. Adjust the user, password, port, and database name to match your PostgreSQL setup.
   - `TESSERACT_CMD` is only needed if Tesseract isn't on your PATH.
   - `OCR_LANG` defaults to `eng`. Use Tesseract language codes for other languages (e.g. `eng+fra`).

5. **Set your ffmpeg path** (MP3 feature only): in `app/main.py`, update the `ffmpeg_location` value to your own ffmpeg `bin` folder, or remove that line if ffmpeg is already on your PATH.

## Running

From the `scanner/` folder:

```
uvicorn app.main:app --host 0.0.0.0 --port 5000 --reload
```

Then open <http://localhost:5000>. The database table is created automatically on first run.

## API

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/documents` | Upload a file and extract its text |
| `GET` | `/api/documents` | List all stored documents |
| `GET` | `/api/documents/{id}` | Get one document, including extracted text |
| `GET` | `/api/documents/{id}/download` | Download the original uploaded file |
| `GET` | `/api/documents/{id}/convert?to=pdf` | Convert to `pdf`, `docx`, `txt`, `png`, or `jpg` |
| `DELETE` | `/api/documents/{id}` | Delete a document |
| `POST` | `/api/url-to-mp3` | Convert a video link to MP3 (JSON body: `{"url": "https://..."}`) |

FastAPI also serves interactive API docs at <http://localhost:5000/docs>.

## Notes

- Uploaded files and generated outputs are stored locally under `scanner/storage/`. Nothing is sent to third parties.
- Only download audio you have the right to download, and respect the terms of the sites you use.
- Text-to-image conversion renders up to 200 lines per image.

## License

No license specified yet. Add one (for example MIT) if you want others to reuse the code.