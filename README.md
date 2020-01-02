# Libao

A Dart wrapper for the [libao](https://www.xiph.org/ao/) library.

Libao is a cross-platform audio library that allows programs to output audio using a simple API on a wide variety of platforms. It currently supports:

* Null output (handy for testing without a sound device)
* WAV files
* AU files
* RAW files
* OSS (Open Sound System, used on Linux and FreeBSD)
* ALSA (Advanced Linux Sound Architecture)
* aRts (Analog RealTime Synth, used by KDE)
* PulseAudio (next generation GNOME sound server)
* esd (EsounD or Enlightened Sound Daemon)
* Mac OS X
* Windows (98 and later)
* AIX
* Sun/NetBSD/OpenBSD
* IRIX
* NAS (Network Audio Server)
* RoarAudio (Modern, multi-OS, networked Sound System)
* OpenBSD's sndio

## Example

```dart
import 'dart:math';
import 'dart:typed_data';

import 'package:libao/libao.dart';

void main() {
  final ao = Libao.open();

  ao.initialize();

  final driverId = ao.defaultDriverId();
  
  print(ao.driverInfo(driverId));

  const bits = 16;
  const channels = 2;
  const rate = 44100;

  final device = ao.openLive(
    driverId,
    bits: bits,
    channels: channels,
    rate: rate,
    matrix: 'R'
  );

  const volume = 0.5;
  const freq = 440.0;

  // Number of bytes * Channels * Sample rate.
  const bufferSize = bits ~/ 8 * channels * rate;
  final buffer = Uint8List(bufferSize);

  for (var i = 0; i < rate; i++) {
    final sample = (volume * 32768.0 * sin(2 * pi * freq * (i / rate))).round();
    // Left = Right.
    buffer[4 * i] = buffer[4 * i + 2] = sample & 0xff;
    buffer[4 * i + 1] = buffer[4 * i + 3] = (sample >> 8) & 0xff;
  }

  ao.play(device, buffer);

  ao.close(device);
  ao.shutdown();
}
```