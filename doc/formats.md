# Supported formats and edge cases

| Format | Detected by | Duration from | Edge cases handled |
|---|---|---|---|
| WAV | `RIFF`/`RF64`/`BW64` + `WAVE` | `data` size ÷ block align ÷ rate (`fact` for compressed audio; without it, an `estimated` byte-rate figure) | `bext`, `iXML`, `JUNK`, `LIST`, `FLLR` … before `fmt `; `data` before `fmt `; odd-sized chunks **with and without** the pad byte; RF64/BW64 `ds64` sizes beyond 4 GiB; `WAVE_FORMAT_EXTENSIBLE`; PCM, float, A-law, µ-law; size 0 / 0xFFFFFFFF from streamed writers; truncated copies; wrong byte-rate fields |
| FLAC | `fLaC` | STREAMINFO total samples ÷ rate | ID3v2 tag(s) in front; total samples 0 = unknown (never zero); full 36-bit sample counts |
| MP3 | A Layer III frame header confirmed by the next frame, after any ID3v2 tags (an ID3v2 tag alone counts only when it runs past the bytes available) | Xing/Info or VBRI frame count | Strict sync rejects PCM and 0xFF runs; MPEG-1/2/2.5; CRC-protected frames; padding after the tag; CBR estimate (flagged) that excludes ID3v1 and APEv2 tags |
| AIFF / AIFF-C | `FORM` + `AIFF`/`AIFC` | `COMM` frames ÷ 80-bit extended rate; for compressed AIFF-C, `COMM` counts packets: × 64 for `ima4`, × 6 for `MAC3`/`MAC6` | Uncompressed AIFF-C (`sowt`, `twos`, `in24`, `fl32`, `fl64` …); `ima4`, MACE, `ulaw`, `alaw`; `SSND` offset; truncated sound data |
| M4A / MP4 / MOV | `ftyp` (image brands excluded) or a leading `moov` | `mvhd` duration ÷ timescale | `moov` at the end; 64-bit box sizes; size-0 boxes; fragmented files via `mehd`; AAC/ALAC codec, rate and channels from the sample entry |

Other compressed AIFF-C types (`GSM `, `ilbc`, `Qclp`, `QDM2` …) are detected
and their codec reported, but get no duration: their `COMM` count can be a
packet count rather than a frame count (it is for `ilbc`), and the factor has
not been confirmed against real files.

Not supported yet (reported as `unknown`): Ogg (Vorbis, Opus, Ogg FLAC,
Speex), raw AAC (ADTS), AMR, CAF, WebM/Matroska, WMA, MP1/MP2. Pull requests
welcome.
