# API guide

The full API reference is on
[pub.dev](https://pub.dev/documentation/hertzhound/latest/). This
page explains the main types and how they fit together.

## Two libraries

| Import | Use it for |
|---|---|
| `package:hertzhound/hertzhound.dart` | Pure Dart, every platform including the web. |
| `package:hertzhound/hertzhound_io.dart` | Adds file helpers; re-exports everything above. |

## Sniffing

Every function returns an `AudioProbe` and never throws on bad bytes
(unknown bytes give `AudioFormat.unknown`). The one exception is I/O:
`sniffFile` and `sniffFileSync` throw a `FileSystemException` when the file
cannot be opened, because a missing file is not an unknown format.

| Function | Source |
|---|---|
| `sniffBytes(bytes, {fileLength})` | Bytes in memory: the whole file, or its start plus its real size. |
| `sniffReaderSync(readAt, {length})` | Your own synchronous random access. |
| `sniffReader(readAt, {length})` | Asynchronous random access, e.g. HTTP range requests. |
| `sniffStream(stream, {length, maxBytes})` | A forward-only stream; stops after `maxBytes` (1 MiB) and cancels it. |
| `sniffFile(path)` / `sniffFileSync(path)` | A file on disk (`dart:io`). Use the sync one in widget tests. |

Reads are bounded: a probe reads a few KiB for most files and never more than
a few MiB, whatever the file claims about itself.

## `AudioProbe`

| Member | Meaning |
|---|---|
| `format` | `AudioFormat.wav`, `flac`, `mp3`, `aiff`, `aifc`, `m4a` or `unknown`. |
| `duration` | From the header, or `null` when the header cannot say. |
| `durationAccuracy` | `exact` (counts the header declares) or `estimated` (an MP3 with no Xing/Info/VBRI header, from its first frame's bitrate; a compressed WAV with no usable `fact`, from its byte rate). |
| `isDurationExact` | `durationAccuracy == exact`. |
| `sampleRate`, `channels`, `bitsPerSample`, `codec` | When the header states them. |
| `audioOffset` | Where the audio payload starts, when known (not reported for M4A/MP4). |
| `notes` | `ProbeNote`s: `id3v2Tag`, `rf64`, `extensibleFormat`, `missingPadByte`, `unknownDataSize`, `truncated`, `fragmented`. |
| `extensionLies(name)` | The name has an extension and it contradicts the bytes. |
| `extensionMatches(name)` | The name's extension leads a player to the right demuxer (false when there is none). |

## `AudioFormat`

`AudioFormat` carries `extension` (`'wav'`), `mimeType` (`'audio/wav'`) and
`extensions`: every name a player maps to the same demuxer, so an `.mp4`
holding AAC is not a lie.

`extensionOf(path)` returns the lowercase extension, ignoring folders,
dotfiles and URL query strings.

## `PlaybackAliases`

`PlaybackAliases` offers `prepare`, `release`, `releasePath`, `owns` and
`clear`; `prepare` reports an `AliasOutcome` (`notNeeded`, `created`,
`unrecognised`, `tooLarge`, `failed`).

`prepare` copies only when the name would mislead the player (wrong or
missing extension), never touches the original, refuses to delete anything
outside its own directory, and caps the copy size (512 MiB by default).
Call `clear()` once at startup to remove copies a crash left behind, and
`release()` when you are done with a track.

## Command-line tool

```sh
dart pub global activate hertzhound
hertzhound ~/Downloads/*.mp3
```

The exit code is 0 when every name tells the truth, 1 if any extension lies,
and 2 if a file cannot be read (or on a usage error).
