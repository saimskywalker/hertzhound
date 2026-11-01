# FAQ

## The symptom, and why it happens

Your Flutter app plays a local audio file and:

- the **duration is wildly wrong** (a six-minute song shows as 46:26), and
- it **plays silence**, or fails with a decoder error,

while the same file plays fine on a desktop. The file is usually a **WAV,
FLAC or M4A wearing an `.mp3` name** (or no extension at all): renamed by a
user, a chat app, a download manager or a content provider.

Why: [`just_audio`](https://pub.dev/packages/just_audio), ExoPlayer/Media3
on Android and AVPlayer on iOS/macOS **let the file extension choose the
demuxer** (Media3 tries the extractor the extension names before any other,
and falls back only if that one declines the bytes). A WAV named `.mp3` goes
to the MP3 extractor, which "finds" frame syncs inside raw PCM, estimates
`size ÷ bitrate` as the duration, and decodes nothing. `ffmpeg` and most
servers sniff the bytes instead, so the bug only shows up on the device.

The extension is a claim; the bytes are the truth. `hertzhound` reads
a few KiB of the file, tells you what it really is, gives you the duration
from the header, and can hand the player a correctly named copy.

## Questions

**Does it decode audio?** No. It reads headers only: no decoder, no plugin,
no platform channel. That is why it is fast and works everywhere.

**How many bytes does it read?** For most files a few KiB. A WAV with large
metadata chunks or an M4A with `moov` at the end needs a few seeks. Reads are
capped per probe no matter what the file claims.

**Why `null` instead of a duration?** When the header cannot say for certain
(FLAC with total samples 0, a WAV whose `data` size was never written and no
file length is known), any number would be a guess. Fall back to the
player's figure.

**Is an MP3 duration exact?** With a Xing/Info or VBRI header it is exact
to the frame (frame count × samples per frame). That count includes the
encoder's delay and padding, so it runs a few tens of milliseconds longer
than the audio itself; the same is true of an AAC M4A whose `mvhd` includes
its priming samples. Without a header it is a bitrate estimate and
`durationAccuracy` says `estimated`; that estimate is wrong for VBR files
that lack a header, which is exactly why it is flagged.

**Should I rename the user's file?** Usually not. Use `PlaybackAliases` to
give the player a correctly named copy and keep the original intact. When
uploading, label the bytes with `probe.format.mimeType` instead of trusting
the name.

**Flutter widget tests hang on `sniffFile`.** Widget tests run on a fake
clock where real asynchronous I/O may never complete. Use `sniffFileSync`
there.

**Does it work on the web?** The core library does; its 64-bit arithmetic is
written to be exact in JavaScript, and CI runs the tests in Chrome. The file
helpers need `dart:io`; on the web, call `sniffBytes` or `sniffStream` on the
picked file's bytes.

## How it compares

Tag-reading packages (for example `audio_metadata_reader`, `metadata_god`,
`flutter_media_metadata`) are about **tags**: title, album, performer, cover
art. Some need a native plugin, and none of them is meant to answer "is this
`.mp3` really an MP3?". Player plugins (`just_audio`, `audioplayers`) report
a duration only after loading, from whichever demuxer the extension picked,
which is exactly the number that is wrong here. MIME-sniffing packages map
magic numbers to a type; they do not read durations, and a check of the
MPEG sync word alone is easy to fool with PCM, which is full of `0xFF` bytes.

`hertzhound` does one thing: **format and duration from the bytes**,
in pure Dart, with no plugin and no decoding. Use it alongside any of the
above.
