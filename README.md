# Debian-based Docker image for GStreamer

Rely on GStreamer [monorepo](https://gitlab.freedesktop.org/gstreamer/gstreamer) to create a Debian bullseye based Docker image with GStreamer, nvidia nvcodec plugin, gstopencv and dlib.

Motivations:
* `nvcodec` plugin is not installed (on Debian) with `apt-get install gstreamer1.0-plugins-bad`
* possibility to choose GStreamer version
* includes opencv and dlib desired versions

The resulting image is published on Docker Hub: https://hub.docker.com/repository/docker/ducksouplab/debian-gstreamer

### Preparation

Download version

```
mkdir -p deps
cd deps
git clone https://gitlab.freedesktop.org/gstreamer/gstreamer.git
```

Download opencv and opencv_contrib src:
```
curl https://github.com/opencv/opencv/archive/refs/tags/4.7.0.zip -L --output deps/opencv.zip
curl https://github.com/opencv/opencv_contrib/archive/refs/tags/4.7.0.zip -L --output deps/opencv_contrib.zip
```

Then edit Dockerfile.multi's `OPENCV_VERSION` env to the chosen version, `4.7.0` in the above example.

Download dlib src:

```
# cd to project root
curl http://dlib.net/files/dlib-19.24.tar.bz2 --output deps/dlib.tar.bz2
```

### Build and run

First of all you may check (in the Dockerfile) that the meson build configuration options fit with your purpose (`-Dbuiltype` for instance). Available options are listed by `meson configure` (within a container)

```
cd deps/gstreamer
git checkout 1.20.5
cd ../..
docker build -f Dockerfile.multi -t debian-gstreamer:latest .
docker run --rm -i -t debian-gstreamer:latest bash
# with GPU
docker run --gpus all --rm -i -t debian-gstreamer:latest bash
```

Create a folder (mounted as a volume in `docker run`), run and enter container, then try nvcodec:

```
mkdir -p data
docker run --gpus all --rm -i -v "$(pwd)"/data:/data -t debian-gstreamer:latest bash
# now in container
gst-launch-1.0 filesrc location=/data/input.mkv ! decodebin ! videoconvert ! nvh264enc ! h264parse ! mp4mux ! filesink location=/data/output.mp4
```

Tag and push image if wanted (`ducksouplab/debian-gstreamer` as an example):

```
docker tag debian-gstreamer:latest ducksouplab/debian-gstreamer:latest
docker push ducksouplab/debian-gstreamer:latest
```

### Build log sample

The meson configuration output ends by listing the enabled build targets, here is a sample of an output log:

```
Build targets in project: 581

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
             ossaudio, oss4, video4linux2, flac, jpeg, png, soup, vpx

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
    Examples        : NO
    Benchmarks      : YES
    Documentation   : NO  disabled
    Orc-test library: YES

All GStreamer modules 1.18.5

Subprojects
    FFmpeg                    : YES 12 warnings
    avtp                      : YES
    dssim                     : YES
    gl-headers                : YES
    gst-devtools              : YES
    gst-editing-services      : YES
    gst-examples              : YES 1 warnings
    gst-integration-testsuites: YES
    gst-libav                 : YES
    gst-omx                   : NO Feature 'omx' disabled
    gst-plugins-bad           : YES 1 warnings
    gst-plugins-base          : YES 2 warnings
    gst-plugins-good          : YES 2 warnings
    gst-plugins-rs            : NO Feature 'rs' disabled
    gst-plugins-ugly          : YES 3 warnings
    gst-python                : NO Feature 'python' disabled
    gst-rtsp-server           : YES
    gstreamer                 : YES 3 warnings
    gstreamer-sharp           : NO Feature 'sharp' disabled
    gstreamer-vaapi           : NO Feature 'vaapi' disabled
    json-glib                 : YES 1 warnings
    libdrm                    : NO
                                Dependency "pciaccess" not found, tried pkgconfig and cmake
    libmicrodns               : YES
    libnice                   : YES
    libopenjp2                : NO
                                Dependency "wxWidgets" not found, tried config-tool
    libpsl                    : YES
    libsoup                   : YES 3 warnings
    libxml2                   : YES
    openh264                  : NO Program 'nasm nasm.exe' not found
    orc                       : YES 3 warnings
    pygobject                 : NO Feature 'python' disabled
    sqlite                    : YES
    tinyalsa                  : NO
                                Neither a subproject directory nor a tinyalsa.wrap file was found.
    x264                      : YES 1 warnings
```

# Old: gst-build

Previously the image was built with [gst-build](https://gitlab.freedesktop.org/gstreamer/gst-build), and its source was downloaded during the build (as opposed to put in `deps` prior to the build).

Building the image was done with:
```
docker build --build-arg gstreamer_tag=1.18.6 -f Dockerfile.bullseye.old -t debian-gstreamer:bullseye-gst1.18.6 .
docker run --gpus all --rm -i -t debian-gstreamer:bullseye-gst1.18.6
```