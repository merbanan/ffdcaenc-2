Introduction
============

dcaenc is an open-source implementation of the DTS Coherent Acoustics codec.
Only the core part of the encoder is implemented, because the public
specification (ETSI TS 102 114 V1.2.1) is insufficient for implementing
anything else.

The DTS technology is heavily patented. Please do not download and do not use 
this software if you live in a country such as the USA where software patents are
legal and you don't have a patent license from DTS, Inc.

The project page is at http://aepatrakov.narod.ru/dcaenc/

The latest code can be obtained from git:

 git clone git://gitorious.org/dtsenc/dtsenc.git

Windows-specific notes
======================

There is a fork of dcaenc specially targetted at Windows users. You can get
it from http://gitorious.org/~mulder/dtsenc/mulders-dtsenc . As compared to
this the original version, that fork adds a Visual Studio project file, a
progress indicator and support for Unicode file names on Windows.
It also breaks Linux support.

ffdcaenc (Free Fast DTS Encoder) is a fork from mulders-dtsenc. It adds
support for 24 bit input files, multiple mono input files, flags for options
needed for DTS CD and DVD, and changes the core dcaenc library to support
strictly-compliant DVD bitrates as expected by AudioMuxer and Muxman.


The command-line application
============================

The ffdcaenc command line utility converts multichannel wav files to DTS (or
one or multiple mono files to DTS). The resulting DTS files can be written to an audio
CD and played via a digital connection (SPDIF or HDMI) to a receiver, or
used as sound tracks for a DVD.

Usage:

 ffdcaenc -i input.wav -o output.dts -b bitrate
 
For 5.1, the input wav file should have the same channel order as defined by SMPTE and ITU,
i.e.: left, right, center, lfe, surround left, surround right.
 
or

 ffdcaenc -m -0 left.wav -1 right.wav -2 center.wav -3 lfe.wav -4 ls.wav -5 rs.wav -o output.dts -b bitrate

using multiple mono wave files. The following mono input file combinations are supported:

	1.0		-2 center.wav
	1.1		-2 center.wav -3 lfe-wav
	2.0		-0 left.wav -1 right.wav
	2.1		-0 left.wav -1 right.wav -3 lfe.wav
	3.0		-0 left.wav -1 right.wav -2 center.wav
	3.1		-0 left.wav -1 right.wav -2 center.wav -3 lfe.wav
	4.0		-0 left.wav -1 right.wav -4 ls.wav -5 rs.wav
	4.1		-0 left.wav -1 right.wav -4 ls.wav -5 rs.wav -3 lfe.wav
	5.0		-0 left.wav -1 right.wav -2 center.wav -4 ls.wav -5 rs.wav
	5.1		-0 left.wav -1 right.wav -2 center.wav -4 ls.wav -5 rs.wav -3 lfe.wav

The files can be in any order on the command line.

Some destinations require a specific bitrate to be specified. To create a
CD-compatible DTS file from a multichannel 44.1kHz sampel rate file, run:

 ffdcaenc -e -r -i input.wav -o output.dts -b 1411.2
 
The dts file will need to be converterd to a wav file in order to be burned to a cd.
This can be done with spdifer, part of the ac3filter projects,
 http://ac3filter.net/

 spdifer output.dts output.wav -wav	

To create a DVD-compatible track from a multichannel (or 6 mono) wav file(s) that has a
48 kHz sample rate:

 ffdcaenc -i input.wav -o output.dts -b 1509.75

or for a half-rate output:

 ffdcaenc -i input.wav -o output.dts -b 754.5

The resulting dts file can be muxed with titles/still images using AudioMuxer,
	http://www.surroundbyus.com/sbu/viewtopic.php?f=7&t=135

or with video files using muxman
	http://stnsoft.com/Muxman/versions.shtml
	
or other tools that support muxing of dts streams.
  
Known bug: wav files with floating-point samples are misinterpreted as
containing 32-bit integer samples.

ORIGINAL DCAENC NOTES
=====================

The original dcaenc can be compiled for Windows, here is how.

The recommended compiler for dcaenc on Windows is MinGW. Get it from
http://sourceforge.net/projects/mingw/files/Installer/ . You may want to use
either the mingw-get.exe utility directly (just copy it to c:\mingw\bin,
install nothing else), or use a graphical installer. In the latter case,
install just the C compiler.

After installation, add c:\mingw\bin and c:\mingw\msys\1.0\bin to the Path
environment variable (right-click My Computer -> Properties -> Advanced ->
Environment Variables) if it is not already there. Then install the required
MinGW components from the command prompt:

 mingw-get install mingw32-base mingw32-autotools

For autotools to work, you also have to copy or rename the
c:\mingw\msys\1.0\etc\fstab.sample file to c:\mingw\msys\1.0\etc\fstab.

Installation
============

The dcaenc library has no external dependencies. The optional ALSA plugin
that comes with it, obviously, depends on the ALSA library compiled with
support for external plugins.

See the file INSTALL for the generic installation instructions.
Quick summary:

 autoreconf -f -i -v       # only if building from git
 ./configure --prefix=/usr # with any other prefix, ALSA won't find the plugin
 make
 su -c 'make install'      # Or, on Ubuntu: sudo make install

Package contents
================

The dcaenc package contains the libdcaenc.so shared library, the corresponding
dcaenc.h header, the dcaenc command line tool, and (optionally) the ALSA
plugin.

The libdcaenc library
=====================

The library may be useful if you want to create DTS streams in your own
application.

To do so, you have to call the following functions:

dcaenc_context dcaenc_create(
    int sample_rate,
    int channel_config,
    int approx_bitrate,
    int flags);

This function creates a library context according to its parameters. The
resulting context is an opaque value that should be passed to the other library
functions as the first parameter.

The sample rate must be one of the following values: 32000, 44100, 48000 or
those divided by 2 or 4. The channel configuration is one of the
DCAENC_CHANNELS_* defines listed in the dcaenc.h header. Note that values
greater than DCAENC_CHANNELS_3FRONT_2REAR always return an error now because
their encoding requires the Xch extension that is undocumented and thus not
implemented. Only non-LFE channels have to be specified here. The approximate
bitrate is specified in bits per second and may be rounded up by the library.

The flags parameter should be a logical OR of zero or more of the
DCAENC_FLAG_* defines. Their meanings:

DCAENC_FLAG_28BIT: use only 28 out of 32 bits in each four bytes of output.
This may be useful if the loudness of the hiss resulting from
mis-interpretation of the encoded stream as PCM must be reduced. However,
this also results in the reduction of the effective bitrate by 12.5%. DTS
CDs are usually encoded with this option.

DCAENC_FLAG_BIGENDIAN: produce a big-endian variant of the DTS bitstream.
This may be useful for writing the stream as a DVD sound track.

DCAENC_FLAG_LFE: indicates that a separate LFE channel has to be added to the
layout indicated by the channel_config parameter.

DCAENC_FLAG_PERFECT_QMF: selects a "perfect-reconstruction" version of the
quadrature mirror filter. This reduces distortions inherent in the filterbank
design, but makes the filter output more sensitive to quantization errors
introduced later in the encoder. libdca version 0.0.5 or earlier will not be
able to decode the resulting stream correctly due to a bug (mistyped table)
in it. It makes no sense to use this option for streams with bitrate of
0.3 Mbps per channel or less.

DCAENC_FLAG_IEC_WRAP: wraps DTS frames as defined by the IEC 61937-5 standard
for transmission over SPDIF. Namely, the library adds a 8-byte header to each
frame and pads the frame with zeroes to achieve the same bitrate as a stereo
16-bit PCM stream.

On any error, dcaenc_create() returns NULL. There is no way to find out the
reason for the error.

int dcaenc_bitrate(dcaenc_context c);

Returns the actual bitrate that is used by the library.

int dcaenc_input_size(dcaenc_context c);

Returns the size of the input buffer that your application has to submit, in
samples. Now this value is always equal to 512.

int dcaenc_output_size(dcaenc_context c);

Returns the size of the output buffer that the application should provide,
in bytes.

int dcaenc_convert_s32(
    dcaenc_context c,
    const int32_t *input,
    uint8_t *output);

Performs the conversion of PCM samples stored in the input buffer to the
DTS bitstream, stores one frame of the encoded bitstream in the output buffer.
The input buffer should contain interleaved signed 32-bit samples. The channel
order is as follows:

DCAENC_CHANNELS_MONO: center
DCAENC_CHANNELS_STEREO: left, right
DCAENC_CHANNELS_3FRONT: center, left, right
DCAENC_CHANNELS_2FRONT_1REAR: left, right, surround
DCAENC_CHANNELS_3FRONT_1REAR: center, left, right, surround
DCAENC_CHANNELS_2FRONT_2REAR: left, right, surround left, surround right
DCAENC_CHANNELS_3FRONT_2REAR: center, left, right, surround left, surround right

If the LFE channel is used, it should be added as the last one.

The following layouts are also defined and do not return an error while
encoding, but do not result in a high-quality bitstream due to incompatibility
with the psychoacoustical model used by the library:

DCAENC_CHANNELS_DUAL_MONO: A, B
DCAENC_CHANNELS_STEREO_SUMDIFF: left+right, left-right
DCAENC_CHANNELS_STEREO_TOTAL: left total, right total

dcaenc_convert_s32() returns the number of bytes written to the output buffer.
Right now, it is always the same as returned by dcaenc_output_size(), but
this will change if variable bitrate encoding is added to the library.

int dcaenc_destroy(dcaenc_context c, uint8_t *output);

Destroys the library context. If a non-NULL value is provided in the output
parameter, the library encodes the final frame and puts it there. This may be
useful because there is a 512-sample latency inherent in the DTS filterbank,
so the output frame gets the last portion of the PCM input submitted earlier.
The returned value indicates the number of bytes written to the output buffer.

ALSA Plugin
===========

The ALSA plugin may be useful for playing multichannel sound from arbitrary
ALSA applications through an SPDIF link. This is needed because the SPDIF
link cannot carry enough bits per second to transport the raw uncompressed
5.1 audio.

The "alsa-plugins" package contains a similar plugin for on-the-fly AC3
encoding.

The plugin should not be used with HDMI connections, unless the video card
imposes the same audio bandwidth limitations as the SPDIF link. The HDMI
standard defines enough bandwidth so that the uncompressed 5.1 PCM stream
fits even at 192 kHz sample rate.

The ALSA plugin should work in real time on any modern CPU. Here on Intel
Core i5 @ 1.20 GHz (i.e. in powersaving mode) it eats ~40% of a single core.

To use the ALSA plugin, add the following line to your $HOME/.asoundrc
file or to /etc/asound.conf:

 <confdir:pcm/dca.conf>

It will create an additional ALSA device for each of your sound cards that
have an SPDIF output. The name of the device will be similar to
"dca:CARD=Intel,DEV=0", or, for the default card, simply "dca".

If you want to encode DTS and send it to something that is not SPDIF, add
a snippet similar to the following:

 pcm.dcahdmi {
     type dca
     slave.pcm "hdmi:CARD=Intel,DEV=0,AES0=6"
 }

The card name can be found in the "aplay -L" output, and AES0=6 parameter
marks the stream as non-audio (so that it doesn't get played by mistake --
this would result in loud hiss that can damage loudspeakers).

Unlike the AC3 encoder, there are no other configuration parameters. This is
because it does not make sense to have them. The hard-coded settings (same
bitrate as stereo PCM, all bits used) provide the best possible quality and
should work for everyone.

To direct mplayer output to the default card via the encoder:

 mplayer -channels 6 -ao alsa:device=dca file.flac

It is not possible to use dmix on top of the encoder. This is a limitation
of dmix: it only works on direct hardware devices providing mmap, and the
dca plugin is not a hardware device and does not provide mmap. Please use
PulseAudio instead.

Known bug: the ALSA plugin doesn't report the supported sample rates correctly.
Fixing this requires rewriting the plugin from the extplug infrastructure to
ioplug. So you may need to add one of the following flags to mplayer command
line:

 -af resample=44100
 -af resample=48000

Use with PulseAudio
===================

The ALSA plugin can be used with PulseAudio. To do so, add the following lines
to the end of the /usr/share/pulseaudio/alsa-mixer/profile-sets/default.conf
file:

[Mapping iec958-dts-surround-51]
description = DTS
device-strings = dca:%f
channel-map = front-left,front-right,rear-left,rear-right,front-center,lfe
priority = 3
direction = output

Later they may be added to the default profile upstream. After restarting
PulseAudio, it will see an additional "DTS" output profile and allow you to
select it in the volume control application such as pavucontrol or
gnome-volume-control.

Compatibility
=============

The author has no DTS-capable receiver and tests the encoder only by decoding
its output with ffmpeg, libdca or ArcSoft DTS decoder (the same engine as
used in WinDVD).

The ALSA plugin has been tested and found to work with the following receivers
by other people:

 Logitech Z5500
 JVC TH-A25
 Samsung HT-Z310

Some receivers (including JVC TH-A25) mute their outputs when receiving full
32-bit DTS stream (as generated by the ALSA plugin) over SPDIF with AES0=6
(the default for the "dca" family of ALSA devices). To overcome this problem,
add the following line to the end of .asoundrc:

 defaults.pcm.dca.aes0 0x04

However, it can cause your receiver to unmute its output even if it does
not support DTS streams or does not detect them reliably. This will result
in very loud hiss that can damage the loudspeakers. So try this setting with
the lowest possible volume, and this is why it is not the default.

Similar settings exist for AES1, AES2 and AES3 SPDIF parameters.

Quality
=======

There are debates on the Internet about the relative quality of AC3 vs DTS.
AC3 uses more advanced compression algorithms, DTS allows for higher bit rates.
There were no blind tests comparing the output of the encoder with anything
else. However, this encoder uses only the most basic compression techniques
defined in the DTS specification, and thus cannot win any comparison with
commercial DTS encoders. Still, at 754 kbps, the internal psychoacoustical
model considers the distortions to be just below the threshold of detection by
human ears.

How to report bugs
==================

Bugs should be reported by email to patrakov@gmail.com, preferrably with a
short (< 10 seconds) flac sample that demonstrates the problem, and a patch
that fixes it.

Thanks
======

The following people helped me to test the encoder (including the versions that
did not work):

 Arun Raghavan

 Colin Guthrie

 Mikhail Elovskikh

 cryptonymous from the linux.org.ru forum

 rulet from the linux.org.ru forum
