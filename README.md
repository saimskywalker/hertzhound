# hertzhound

Find the real format and duration of an audio file from its bytes, so a WAV,
FLAC or M4A with an `.mp3` name stops breaking playback in your Dart or
Flutter app.

[![pub version](https://img.shields.io/pub/v/hertzhound)](https://pub.dev/packages/hertzhound)
[![pub points](https://img.shields.io/pub/points/hertzhound)](https://pub.dev/packages/hertzhound/score)
[![CI](https://github.com/saimskywalker/hertzhound/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/saimskywalker/hertzhound/actions/workflows/ci.yml)
[![license: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/saimskywalker/hertzhound/blob/main/LICENSE)

**Status:** in maintenance. Bug reports and pull requests are still reviewed; new features are on hold.

## Why

- A local file shows a wildly wrong duration (a six-minute song as 46:26) or
  plays silence on the device, while it plays fine on a desktop.
- The file is a WAV, FLAC or M4A with an `.mp3` name, or no extension.
  `just_audio`, ExoPlayer/Media3 and AVPlayer let the extension choose the
  demuxer, so the wrong one reads it
  ([details](https://github.com/saimskywalker/hertzhound/blob/main/doc/faq.md#the-symptom-and-why-it-happens)).
- You want the truth from a few KiB of the file: pure Dart, no
  dependencies, on Android, iOS, macOS, Windows, Linux, the web, servers
  and CLIs.

## Install

```sh
dart pub add hertzhound    # or: flutter pub add hertzhound
```

## Quick start

```dart
import 'package:hertzhound/hertzhound_io.dart';

final probe = await sniffFile('/path/to/take_03.mp3');

probe.format;                       // AudioFormat.wav
probe.duration;                     // 0:06:00.000000, from the header
probe.isDurationExact;              // true
probe.extensionLies('take_03.mp3'); // true: it is really a .wav
```

For bytes you already hold (works on the web too), call
`sniffBytes(firstBytes, fileLength: totalSize)` from
`package:hertzhound/hertzhound.dart`.

### Fix playback in Flutter with just_audio

```dart
import 'package:hertzhound/hertzhound_io.dart';
import 'package:just_audio/just_audio.dart';
import 'package:path_provider/path_provider.dart';

final aliases = PlaybackAliases(
  '${(await getTemporaryDirectory()).path}/hertzhound',
);
await aliases.clear(); // once at startup: removes copies a crash left behind

final playable = await aliases.prepare(pickedPath, name: pickedDisplayName);
await player.setFilePath(playable.path);
// Prefer the header's duration; fall back to the player's.
final duration = playable.probe.duration ?? player.duration;
// ...when done with this track:
await aliases.release(playable);
```

`prepare` copies only when the name would mislead the player, never touches
the original, and caps the copy size (512 MiB by default).

## Example output

The command-line tool (`dart pub global activate hertzhound`), run
on synthetic test files with misleading names:

```text
$ hertzhound take_03.mp3 voice_note.mp3 song.mp3 interview.m4a
take_03.mp3
  format:   wav / pcm
  duration: 0:00:00.250000 (exact)
  WARNING:  the extension lies; this is a .wav file

voice_note.mp3
  format:   flac / flac
  duration: 0:00:03.500000 (exact)
  WARNING:  the extension lies; this is a .flac file

song.mp3
  format:   mp3 / mp3
  duration: 0:00:00.992653 (exact)
  notes:    id3v2Tag

interview.m4a
  format:   m4a / alac
  duration: 0:00:07.000000 (exact)
```

The exit code is 1 when any extension lies and 2 when a file cannot be read.

## What it reads

| Format | Duration from |
|---|---|
| WAV, RF64, BW64 | `data` size (or `fact`) and the format chunk |
| FLAC | STREAMINFO total samples |
| MP3 | Xing/Info or VBRI frame count; otherwise a flagged bitrate estimate |
| AIFF / AIFF-C | `COMM` frames (packets for `ima4` and MACE) |
| M4A / MP4 / MOV | `mvhd`, including `moov` at the end and fragmented files |

- Header parsing only: no decoder, no plugin, no platform channel.
- Never throws on bad bytes; unknown bytes give `AudioFormat.unknown`.
- Bounded reads: a few KiB for most files, never more than a few MiB.
- A duration it cannot be sure of is `null` or marked `estimated`, never a
  guess reported as exact.
- Not supported yet: Ogg, raw AAC (ADTS), AMR, CAF, WebM/Matroska, WMA,
  MP1/MP2.

## Coming from imaji_audio_sniff

`hertzhound` is the same code under a new name; the API is otherwise
unchanged. To move over, replace the dependency and update these names:

| imaji_audio_sniff | hertzhound |
|---|---|
| `package:imaji_audio_sniff/imaji_audio_sniff.dart` | `package:hertzhound/hertzhound.dart` |
| `package:imaji_audio_sniff/imaji_audio_sniff_io.dart` | `package:hertzhound/hertzhound_io.dart` |
| CLI `imaji_audio_sniff` | CLI `hertzhound` |
| `PlaybackAliases.inSystemTemp` default folder `imaji_audio_sniff_playback` | `hertzhound_playback` |
| Playback copies named `imaji-<run>-<n>.<ext>` | `hertzhound-<run>-<n>.<ext>` |

`PlaybackAliases.clear()` removes only files with the new prefix, so delete
an old folder's leftover copies yourself (or point the new instance at a new
folder).

## Documentation

- [API guide](https://github.com/saimskywalker/hertzhound/blob/main/doc/api.md): every sniff function, `AudioProbe`, `AudioFormat`, `PlaybackAliases`, the CLI
- [Supported formats and edge cases](https://github.com/saimskywalker/hertzhound/blob/main/doc/formats.md)
- [FAQ](https://github.com/saimskywalker/hertzhound/blob/main/doc/faq.md): why players get it wrong, accuracy, widget tests, the web, how it compares

## Links

- [pub.dev page](https://pub.dev/packages/hertzhound)
- [API reference](https://pub.dev/documentation/hertzhound/latest/)
- [CHANGELOG](https://github.com/saimskywalker/hertzhound/blob/main/CHANGELOG.md)
- [Issues](https://github.com/saimskywalker/hertzhound/issues)
- [Contributing](https://github.com/saimskywalker/hertzhound/blob/main/CONTRIBUTING.md)

## License

MIT © saimskywalker. See [LICENSE](https://github.com/saimskywalker/hertzhound/blob/main/LICENSE).
