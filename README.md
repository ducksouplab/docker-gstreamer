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

To build with image layers cache and save the output of the build you may alternatively run:

```
docker build --no-cache -f Dockerfile.multi -t debian-gstreamer:latest . 2>&1 | tee build.log
```

Create a data folder (mounted as a volume in `docker run`) with an input.mkv file , run and enter container, then try nvcodec:

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
Build targets in project: 493

libdv 1.0.0

    YUV format            : YUY2
    assembly optimizations: YES

gst-editing-services 1.22.0

    Plugins: nle, ges

gst-plugins-bad 1.22.0

    Plugins               : accurip, adpcmdec, adpcmenc, aiff, asfmux,
                            audiobuffersplit, audiofxbad, audiomixmatrix,
                            audiolatency, audiovisualizers, autoconvert, bayer,
                            camerabin, codecalpha, codectimestamper,
                            coloreffects, debugutilsbad, dvbsubenc,
                            dvbsuboverlay, dvdspu, faceoverlay, festival,
                            fieldanalysis, freeverb, frei0r, gaudieffects, gdp,
                            geometrictransform, id3tag, inter, interlace,
                            ivfparse, ivtc, jp2kdecimator, jpegformat, rfbsrc,
                            midi, mpegpsdemux, mpegpsmux, mpegtsdemux,
                            mpegtsmux, mxf, netsim, rtponvif, pcapparse, pnm,
                            proxy, legacyrawparse, removesilence, rist, rtmp2,
                            rtpmanagerbad, sdpelem, segmentclip, siren, smooth,
                            speed, subenc, switchbin, timecode, transcode,
                            videofiltersbad, videoframe_audiolevel,
                            videoparsersbad, videosignal, vmnc, y4mdec,
                            decklink, dvb, fbdevsink, ipcpipeline, nvcodec,
                            qsv, shm, va, aes, avtp, closedcaption, dash, dtls,
                            fdkaac, hls, iqa, microdns, opencv, opusparse,
                            sctp, smoothstreaming, sndfile, soundtouch,
                            ttmlsubs, webrtc
    (A)GPL license allowed: True

gst-plugins-base 1.22.0

    Plugins: adder, app, audioconvert, audiomixer, audiorate, audioresample,
             audiotestsrc, compositor, encoding, gio, overlaycomposition,
             pbtypes, playback, rawparse, subparse, tcp, typefindfunctions,
             videoconvertscale, videorate, videotestsrc, volume, ogg, opus,
             pango, vorbis, ximagesink

gst-plugins-good 1.22.0

    Plugins: alpha, alphacolor, apetag, audiofx, audioparsers, auparse,
             autodetect, avi, cutter, navigationtest, debug, deinterlace, dtmf,
             effectv, equalizer, flv, flxdec, goom, goom2k1, icydemux,
             id3demux, imagefreeze, interleave, isomp4, alaw, mulaw, level,
             matroska, monoscope, multifile, multipart, replaygain, rtp,
             rtpmanager, rtsp, shapewipe, smpte, spectrum, udp, videobox,
             videocrop, videofilter, videomixer, wavenc, wavparse, xingmux,
             y4menc, ossaudio, oss4, video4linux2, ximagesrc, adaptivedemux2,
             cairo, flac, jpeg, lame, dv, png, soup, vpx

gst-plugins-ugly 1.22.0

    Plugins               : asf, dvdlpcmdec, dvdsub, realmedia, x264
    (A)GPL license allowed: True

gst-rtsp-server 1.22.0

    Plugins: rtspclientsink

gstreamer 1.22.0

    Plugins: coreelements, coretracers

gstreamer-vaapi 1.22.0

    Plugins: vaapi

json-glib 1.6.6

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

gstreamer-full 1.22.0

  Build options
    gstreamer-full library    : NO
    Tools                     : gst-inspect, gst-stats, gst-typefind,
                                gst-launch, gst-device-monitor, gst-discoverer,
                                gst-play, ges-launch

  Subprojects
    FFmpeg                    : YES
    avtp                      : YES
    dssim                     : YES
    dv                        : YES
    fdk-aac                   : YES
    gl-headers                : YES
    gst-devtools              : NO Feature 'devtools' disabled
    gst-editing-services      : YES
    gst-examples              : YES 3 warnings
    gst-integration-testsuites: NO Feature 'devtools' disabled
    gst-libav                 : YES
    gst-omx                   : NO Feature 'omx' disabled
    gst-plugins-bad           : YES 1 warnings
    gst-plugins-base          : YES 1 warnings
    gst-plugins-good          : YES 1 warnings
    gst-plugins-rs            : NO Feature 'rs' disabled
    gst-plugins-ugly          : YES
    gst-python                : NO
                                Subproject "subprojects/pygobject" required but not found.
    gst-rtsp-server           : YES
    gstreamer                 : YES 1 warnings
    gstreamer-sharp           : NO Feature 'sharp' disabled
    gstreamer-vaapi           : YES
    json-glib                 : YES 1 warnings
    lame                      : YES 1 warnings
    libdrm                    : NO
                                Dependency "pciaccess" not found, tried pkgconfig and cmake
    libjpeg-turbo             : YES
    libmicrodns               : YES
    libnice                   : YES
    libopenjp2                : NO
                                In subproject libopenjp2: Unknown options: "libopenjp2:build_codec"
    libpsl                    : YES
    libsoup                   : NO
                                Assert failed: libsoup requires glib-networking for TLS support
    libxml2                   : YES
    openh264                  : NO Program 'nasm' not found or not executable
    pygobject                 : NO
                                Dependency 'glib-2.0' is required but not found.
    sqlite3                   : YES
    tinyalsa                  : NO
                                Neither a subproject directory nor a tinyalsa.wrap file was found.
    x264                      : YES 1 warnings

  User defined options
    buildtype                 : release
    prefix                    : /gstreamer/install
    devtools                  : disabled
    doc                       : disabled
    examples                  : disabled
    gpl                       : enabled
    libav                     : enabled
    orc                       : disabled
    rs                        : disabled
    tests                     : disabled
    vaapi                     : enabled
    gst-plugins-base:pango    : enabled
    gst-plugins-ugly:x264     : enabled
    libsoup:sysprof           : disabled
```
