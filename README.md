# Debian-based Docker image for GStreamer

Rely on [gst-build](https://gitlab.freedesktop.org/gstreamer/gst-build) to create a Debian bullseye based Docker image with GStreamer, nvidia nvcodec plugin, gstopencv and dlib.

Motivations:
* `nvcodec` plugin is not installed (on Debian) with `apt-get install gstreamer1.0-plugins-bad`
* possibility to choose GStreamer version
* includes opencv and dlib

The resulting image is published on Docker Hub: https://hub.docker.com/repository/docker/creamlab/bullseye-gstreamer

### Preparation

Download dlib src:

```
mkdir -p deps
curl http://dlib.net/files/dlib-19.22.tar.bz2 --output deps/dlib.tar.bz2
```

### Build and run

```
docker build --build-arg gstreamer_tag=1.18.5 -f Dockerfile.bullseye -t bullseye-gstreamer .
```

Tag and push:

```
docker tag bullseye-gstreamer creamlab/bullseye-gstreamer
docker push creamlab/bullseye-gstreamer
```

Enter the container and try nvcodec:

```
mkdir -p data
docker run --gpus all --rm -i -v "$(pwd)"/data:/data -t bullseye-gstreamer:latest bash
# now in container
gst-launch-1.0 filesrc location=/data/input.mkv ! decodebin ! videoconvert ! nvh264enc ! h264parse ! mp4mux ! filesink location=/data/output.mp4
```

### Build log sample

The meson configuration output ends by listing the enabled build targets, here is a sample of an output log:

```
Message: Building subprojects: gstreamer, gst-plugins-base, gst-plugins-good, libnice, gst-plugins-bad, gst-plugins-ugly, gst-libav, gst-rtsp-server, gst-devtools, gst-integration-testsuites, gst-editing-services, pygobject, gst-python, gst-examples
Program gst-env.py found: YES (/gstreamer/src/gst-env.py)
Program git-update found: YES (/gstreamer/src/git-update)
Build targets in project: 792

gst-plugins-bad 1.18.5

    Plugins: accurip, adpcmdec, adpcmenc, aiff, asfmux, audiobuffersplit,
             audiofxbad, audiomixmatrix, audiolatency, audiovisualizers,
             autoconvert, bayer, camerabin, coloreffects, debugutilsbad,
             dvbsubenc, dvbsuboverlay, dvdspu, faceoverlay, festival,
             fieldanalysis, freeverb, frei0r, gaudieffects, gdp,
             geometrictransform, id3tag, inter, interlace, ivfparse, ivtc,
             jp2kdecimator, jpegformat, rfbsrc, midi, mpegpsdemux, mpegpsmux,
             mpegtsdemux, mpegtsmux, mxf, netsim, rtponvif, pcapparse, pnm,
             proxy, legacyrawparse, removesilence, rist, rtmp2, rtpmanagerbad,
             sdpelem, segmentclip, siren, smooth, speed, subenc, switchbin,
             timecode, transcode, videofiltersbad, videoframe_audiolevel,
             videoparsersbad, videosignal, vmnc, y4mdec, decklink, dvb,
             fbdevsink, ipcpipeline, nvcodec, shm, avtp, dash, dc1394, dtls,
             hls, iqa, microdns, opencv, openexr, opusparse, sctp,
             smoothstreaming, sndfile, soundtouch, webrtc

gst-plugins-base 1.18.5

    Plugins: adder, app, audioconvert, audiomixer, audiorate, audioresample,
             audiotestsrc, compositor, encoding, gio, overlaycomposition,
             pbtypes, playback, rawparse, subparse, tcp, typefindfunctions,
             videoconvert, videorate, videoscale, videotestsrc, volume, ogg,
             opus, vorbis

gst-plugins-good 1.18.5

    Plugins: alpha, alphacolor, apetag, audiofx, audioparsers, auparse,
             autodetect, avi, cutter, navigationtest, debug, deinterlace, dtmf,
             effectv, equalizer, flv, flxdec, goom, goom2k1, icydemux,
             id3demux, imagefreeze, interleave, isomp4, alaw, mulaw, level,
             matroska, monoscope, multifile, multipart, replaygain, rtp,
             rtpmanager, rtsp, shapewipe, smpte, spectrum, udp, videobox,
             videocrop, videofilter, videomixer, wavenc, wavparse, y4menc,
             ossaudio, oss4, video4linux2, flac, jpeg, png, soup

gst-plugins-ugly 1.18.5

    Plugins: asf, dvdlpcmdec, dvdsub, realmedia, xingmux, x264

gstreamer 1.18.5

    Plugins: coreelements, coretracers

json-glib 1.6.7

  Directories
    prefix       : /gstreamer/install
    includedir   : /gstreamer/install/include
    libdir       : /gstreamer/install/lib/x86_64-linux-gnu
    datadir      : /gstreamer/install/share

  Build
    Introspection: YES
    Documentation: NO
    Manual pages : NO
    Tests        : YES

orc 0.4.32

  Backends
    SSE             : YES
    MMX             : YES
    NEON            : YES
    MIPS            : YES
    c64x            : YES
    Altivec         : YES

  Build options
    Tools           : YES
    Tests           : NO
    Examples        : YES
    Benchmarks      : YES
    Documentation   : NO  disabled
    Orc-test library: YES

All GStreamer modules 1.18.5

  Subprojects
    FFmpeg                    : YES 12 warnings
    avtp                      : YES
    dssim                     : YES
    gl-headers                : YES
    gobject-introspection     : YES 3 warnings
    gst-devtools              : YES
    gst-editing-services      : YES
    gst-examples              : YES
    gst-integration-testsuites: YES
    gst-libav                 : YES
    gst-omx                   : NO Feature 'omx' disabled
    gst-plugins-bad           : YES 1 warnings
    gst-plugins-base          : YES 2 warnings
    gst-plugins-good          : YES 2 warnings
    gst-plugins-rs            : NO Feature 'rs' disabled
    gst-plugins-ugly          : YES 3 warnings
    gst-python                : YES
    gst-rtsp-server           : YES
    gstreamer                 : YES 3 warnings
    gstreamer-sharp           : NO Feature 'sharp' disabled
    gstreamer-vaapi           : NO Feature 'vaapi' disabled
    json-glib                 : YES 1 warnings
    libdrm                    : NO
                                Dependency "pciaccess" not found, tried pkgconfig
    libmicrodns               : YES
    libnice                   : YES
    libopenjp2                : NO
                                Dependency "wxWidgets" not found, tried config-tool
    libpsl                    : YES
    libsoup                   : YES 3 warnings
    libxml2                   : YES
    openh264                  : NO Program 'nasm nasm.exe' not found
    orc                       : YES 3 warnings
    pycairo                   : NO
                                Dependency "cairo" not found, tried pkgconfig
    pygobject                 : YES 1 warnings
    sqlite                    : YES
    tinyalsa                  : NO
                                Neither a subproject directory nor a tinyalsa.wrap file was found.
    x264                      : YES

```