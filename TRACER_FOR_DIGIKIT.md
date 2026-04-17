# Tracer — Invisible Watermark Engine
### Built for DigiKit

---

## The Problem

AI image manipulation tools are becoming increasingly accessible. Anyone can take a campaign image your agency produces, alter what it says, strip your branding, or redistribute it without credit — and there is currently no reliable way to prove the original source, ownership, or distribution context of a digital image once it leaves your hands.

**Tracer solves this.**

---

## What is Tracer?

Tracer is a cryptography-based signal processing engine that embeds an **invisible watermark** directly into the pixel data of any campaign image. The watermark is completely imperceptible to the human eye — the image looks identical before and after — but it permanently encodes structured metadata about the image's origin.

At any point in the future, that image can be uploaded to Tracer and the hidden data can be read back out, proving:

- Which agency produced it
- Which client it was made for
- Which campaign it belongs to
- The exact date and time it was marked
- Any additional notes attached at the time of creation

---

## Features

### Embed Watermark
- Upload any campaign image (PNG, JPG, WEBP — minimum 256 × 256 px)
- Fill in campaign metadata: agency name, client name, campaign name, date, time, and an optional note
- Tracer encodes a **1024-bit payload** invisibly into the image's frequency data
- Download the watermarked image — it is visually identical to the original
- The entire process completes in under 2 seconds on standard hardware

### Decode Watermark
- Upload any Tracer-watermarked image
- Tracer scans the image's frequency layers and extracts the hidden payload
- All embedded metadata is displayed instantly: agency, client, campaign, date, time, note
- Works on any copy of the image that has not been resized or rotated

### Watermark Payload Structure
Each watermark stores the following fields, packed into a fixed 128-byte binary payload:

| Field    | Max Length | Example              |
|----------|------------|----------------------|
| Agency   | 20 chars   | DigiKit              |
| Client   | 20 chars   | Acme Corporation     |
| Campaign | 20 chars   | Summer 2026          |
| Date     | 8 chars    | 20260412             |
| Time     | 6 chars    | 143000               |
| Note     | 31 chars   | Approved final asset |

The payload begins with a `TRC1` magic header, making it uniquely identifiable as a Tracer watermark.

### Resistance & Durability
The watermark survives the following common image operations:

| Attack              | Survives? |
|---------------------|-----------|
| JPEG compression    | Yes       |
| Colour filter / LUT | Yes       |
| Brightness / contrast adjustment | Yes |
| Noise overlay       | Yes       |
| Screenshot / re-save | Yes      |
| Resize              | No        |
| Rotation > 5°       | No        |

> This means any image shared online, screenshotted, re-uploaded, or run through a colour grade can still be traced back to its origin. The watermark only breaks under geometric transformation (resize / rotate), which would also visibly degrade the image itself.

---

## How It Works

Tracer uses a three-stage frequency-domain encoding process:

**1 — DWT Decomposition**
The image is decomposed into frequency sub-bands using a **Discrete Wavelet Transform** (Haar basis). This separates the image into low-frequency content (shapes, large colour fields) and high-frequency content (edges, fine detail).

**2 — DCT + SVD Bit Injection**
Within the wavelet sub-bands, the image is divided into 4 × 4 coefficient blocks. Each block undergoes a **Discrete Cosine Transform** followed by **Singular Value Decomposition**. The watermark bits are encoded into the dominant singular values of each block — a region of the data that is perceptually invisible but mathematically stable.

**3 — Inverse Reconstruction**
The modified coefficients are passed back through inverse DCT and inverse DWT, reconstructing a pixel image that is visually indistinguishable from the original but now carries the full 1024-bit payload embedded in its structure.

To decode, the same decomposition is applied in reverse — the singular values are read, the bits are extracted, and the payload is reconstructed and parsed.

---

## Technical Stack

| Layer     | Technology                          |
|-----------|-------------------------------------|
| Backend   | Python 3 · Flask                    |
| Watermark Engine | DWT + DCT + SVD (via OpenCV, PyWavelets, NumPy) |
| Frontend  | HTML · CSS · Vanilla JavaScript     |
| Image I/O | OpenCV (cv2)                        |
| Encoding  | `dwtDctSvd` method — proven accurate on real-world images |

The system runs entirely locally — no image data is sent to any external server. Everything is processed on-machine and returned directly to the browser.

---

## API Endpoints

The backend exposes two simple REST endpoints, making Tracer easy to integrate into any existing workflow or platform:

```
POST /api/embed
  Form fields: image (file), company, client, campaign, date, time, note
  Returns:     watermarked PNG file (download)

POST /api/decode
  Form fields: image (file)
  Returns:     JSON with { company, client, campaign, date, time, note }
```

---

## Running the Project

**Requirements**
- Python 3.8 or higher
- pip

**Install dependencies**
```bash
cd tracer_app
pip install -r requirements.txt
```

**Start the server**
```bash
python3 server.py
```

**Open the app**
```
http://localhost:5050
```

---

## Use Cases for DigiKit

- **Proof of authorship** — If a campaign image is reused without permission, Tracer proves DigiKit created it and when.
- **AI manipulation detection** — If an image is AI-morphed and redistributed, the original source can be verified by decoding the unaltered version DigiKit holds.
- **Client delivery tracking** — Every image delivered to a client carries an invisible record of the campaign, date, and handoff. No more disputes over which version was approved.
- **Brand protection** — Watermarked images carry DigiKit's identity invisibly. Even if an image is stripped of visible branding, the watermark remains.
- **Audit trail** — Build a library of watermarked assets. Every image in your archive can be decoded at any time to confirm its origin and intended use.

---

## Project Structure

```
Tracer/
├── tracer_app/
│   ├── server.py              # Flask API backend
│   ├── requirements.txt       # Python dependencies
│   └── templates/
│       └── index.html         # Frontend UI
└── invisible-watermark/       # Watermark engine (DWT+DCT+SVD)
    └── imwatermark/
        ├── watermark.py
        ├── dwtDctSvd.py
        └── maxDct.py
```

---

*Tracer — built for DigiKit · 2026*
