# Debian-based Docker image for GStreamer

Currently this project builds a Debian 12 image with:

- GStreamer 1.22.6
- opencv and opencv_contrib 4.8.0
- dlib 19.24
- libnvrtc (from CUDA) 12.2

It's available here: https://hub.docker.com/r/ducksouplab/debian-gstreamer

It relies on GStreamer [monorepo](https://gitlab.freedesktop.org/gstreamer/gstreamer) to build GStreamer and its plugins (in particular enabling `nvcodec`).

Motivations:
* possibility to choose GStreamer (latest) version
* `nvcodec` plugin is not installed (on Debian) with `apt-get install gstreamer1.0-plugins-bad`
* includes opencv and dlib

### Preparation: download dependencies

Prepare `deps` folder to store source and binaries that will be used by the Dockerfile:

```
mkdir -p deps
cd deps
```

You will have to edit Dockerfile.debian12 if you select different versions of opencv, dlib or nvrtc.

#### GStreamer

Download source and choose version:

```
git clone https://gitlab.freedesktop.org/gstreamer/gstreamer.git
cd gstreamer
git pull
git checkout 1.22.6
cd ..
```

#### opencv

Download opencv and opencv_contrib src:
```
curl https://github.com/opencv/opencv/archive/refs/tags/4.8.0.zip -L --output deps/opencv-4.8.0.zip
curl https://github.com/opencv/opencv_contrib/archive/refs/tags/4.8.0.zip -L --output deps/opencv_contrib-4.8.0.zip
```

Then edit Dockerfiles `OPENCV_VERSION` env to the chosen version, `4.8.0` in the above example.

#### dlib

Download dlib src:

```
# cd to project root
curl http://dlib.net/files/dlib-19.24.tar.bz2 --output deps/dlib.tar.bz2
```

#### nvrtc

libnvrtc.so is needed to enable `cudaconvert`, `cudascale `and `cudaconvertscale` GStreamer plugins.

In the following example we picked CUDA 12.2 that is compatible with the driver version 525.

```
wget https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/cuda-nvrtc-dev-12-2_12.2.128-1_amd64.deb
wget https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/cuda-nvrtc-12-2_12.2.128-1_amd64.deb
```

You may pick another, looking at: https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/

Alternatives to this handpicked nvrtc package installation are:

- install the whole CUDA toolkit (https://developer.nvidia.com/cuda-downloads for CUDA 12 or https://developer.nvidia.com/cuda-11-7-0-download-archive for CUDA 11.7)

- start from an nvidia/cuda devel image (rather than a bare debian one), for instance nvidia/cuda:11.7.0-devel-ubuntu22.04 (`Dockerfile.ubuntu.cuda` is provided as an example)


### Build and run

First of all you may check (in the Dockerfile) that the meson build configuration options fit with your purpose (`-Dbuiltype` for instance). Available options are listed by `meson configure` (within a container).


#### Without plugins
To build with image layers cache and save the output of the build you may:

```
docker build -f Dockerfile.debian12 -t debian-gstreamer:deb12-cuda12.2-gst1.22.6 . 2>&1 | tee build.log
```

Without the cache:

```
# from the root project folder
docker build --no-cache -f Dockerfile.debian12 -t debian-gstreamer:deb12-cuda12.2-gst1.22.6 . 2>&1 | tee build.debian12.log

# run container
docker run --rm -i -t debian-gstreamer:deb12-cuda12.2-gst1.22.6 bash

# with GPU
docker run --gpus all --rm -i -t debian-gstreamer:deb12-cuda12.2-gst1.22.6 bash

# within the container
gst-inspect-1.0 nvcodec
```

Create a data folder (mounted as a volume in `docker run`) with an input.mkv file , run and enter container, then try nvcodec:

```
mkdir -p data
docker run --gpus all --rm -i -v "$(pwd)"/data:/data -t debian-gstreamer:deb12-cuda12.2-gst1.22.6 bash
# now in container
gst-launch-1.0 filesrc location=/data/input.mkv ! decodebin ! videoconvert ! nvh264enc ! h264parse ! mp4mux ! filesink location=/data/output.mp4
```

Share image:

Tag and push image if wanted (`ducksouplab/debian-gstreamer` as an example):

```
docker tag debian-gstreamer:deb12-cuda12.2-gst1.22.6 ducksouplab/debian-gstreamer:deb12-cuda12.2-gst1.22.6
docker push ducksouplab/debian-gstreamer:deb12-cuda12.2-gst1.22.6
```

#### Build with Plugins
To build with image layers cache and save the output of the build you may:

```
docker build -f Dockerfile.plugins.debian12 -t debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6 . 2>&1 | tee build.log
```

Without the cache:

```
# from the root project folder
docker build --no-cache -f Dockerfile.plugins.debian12 -t debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6 . 2>&1 | tee build.debian12.log

# run container
docker run --rm -i -t debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6 bash

# with GPU
docker run --gpus all --rm -i -t debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6 bash

# within the container
gst-inspect-1.0 nvcodec
```

Create a data folder (mounted as a volume in `docker run`) with an input.mkv file , run and enter container, then try nvcodec:

```
mkdir -p data
docker run --gpus all --rm -i -v "$(pwd)"/data:/data -t debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6 bash
# now in container
gst-launch-1.0 filesrc location=/data/input.mkv ! decodebin ! videoconvert ! nvh264enc ! h264parse ! mp4mux ! filesink location=/data/output.mp4
```

Share image:

Tag and push image if wanted (`ducksouplab/debian-gstreamer` as an example):

```
docker tag debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6 ducksouplab/debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6
docker push ducksouplab/debian-gstreamer:deb12-with-plugins-cuda12.2-gst1.22.6
```


### Build from CUDA image

```
docker build -f Dockerfile.ubuntu.cuda -t ubuntu-cuda-gstreamer:ubuntu22.04-cuda11.7.0-gstreamer1.22.6 .
```

Share
```
docker tag ubuntu-cuda-gstreamer:ubuntu22.04-cuda11.7.0-gstreamer1.22.6 ducksouplab/ubuntu-cuda-gstreamer:ubuntu22.04-cuda11.7.0-gstreamer1.22.6
docker push ducksouplab/ubuntu-cuda-gstreamer:ubuntu22.04-cuda11.7.0-gstreamer1.22.6
```

#### Build for ARM processors
To build for ARM (e.g. macs, without CUDA), use the Dockerfile.plugins.arm.debian12 dockerfile

```
docker build -f Dockerfile.plugins.arm.debian12 -t debian-gstreamer:deb12-with-plugins-gst1.22.6-arm . 2>&1 | tee build.log
docker tag debian-gstreamer:deb12-with-plugins-gst1.22.6-arm ducksouplab/debian-gstreamer:deb12-with-plugins-gst1.22.6-arm
docker push ducksouplab/debian-gstreamer:deb12-with-plugins-gst1.22.6-arm
```


#### Build for Intel processors - CPU ONLY
To build for Intel computers (e.g. macs without CUDA), use the Dockerfile.plugins.debian12.intel.cpu_only dockerfile

```
docker build -f Dockerfile.plugins.debian12.intel.cpu_only -t debian-gstreamer:deb12-with-plugins-gst1.22.6-intel-cpu-only . 2>&1 | tee build.log
docker tag debian-gstreamer:deb12-with-plugins-gst1.22.6-intel-cpu-only ducksouplab/debian-gstreamer:deb12-with-plugins-gst1.22.6-intel-cpu-only
docker push ducksouplab/debian-gstreamer:deb12-with-plugins-gst1.22.6-intel-cpu-only
```




### Image build log sample

The meson configuration output ends by listing the enabled build targets, here is a sample of an output log:

```
Build targets in project: 411
 
 libdv 1.0.0
 
     YUV format            : YUY2
     assembly optimizations: YES
 
 gst-editing-services 1.22.6
 
     Plugins: nle, ges
 
 gst-plugins-bad 1.22.6
 
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
 
 gst-plugins-base 1.22.6
 
     Plugins: adder, app, audioconvert, audiomixer, audiorate, audioresample,
              audiotestsrc, compositor, encoding, gio, overlaycomposition,
              pbtypes, playback, rawparse, subparse, tcp, typefindfunctions,
              videoconvertscale, videorate, videotestsrc, volume, ogg, opus,
              pango, vorbis, ximagesink
 
 gst-plugins-good 1.22.6
 
     Plugins: alpha, alphacolor, apetag, audiofx, audioparsers, auparse,
              autodetect, avi, cutter, navigationtest, debug, deinterlace, dtmf,
              effectv, equalizer, flv, flxdec, goom, goom2k1, icydemux,
              id3demux, imagefreeze, interleave, isomp4, alaw, mulaw, level,
              matroska, monoscope, multifile, multipart, replaygain, rtp,
              rtpmanager, rtsp, shapewipe, smpte, spectrum, udp, videobox,
              videocrop, videofilter, videomixer, wavenc, wavparse, xingmux,
              y4menc, ossaudio, oss4, video4linux2, ximagesrc, adaptivedemux2,
              cairo, flac, jpeg, lame, dv, png, soup, vpx
 
 gst-plugins-ugly 1.22.6
 
     Plugins               : asf, dvdlpcmdec, dvdsub, realmedia, x264
     (A)GPL license allowed: True
 
 gst-rtsp-server 1.22.6
 
     Plugins: rtspclientsink
 
 gstreamer 1.22.6
 
     Plugins: coreelements, coretracers
 
 gstreamer-vaapi 1.22.6
 
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
 
 gstreamer-full 1.22.6
 
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
                                 Dependency 'glib-2.0' is required but not found.
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
     libxml2                   : YES 2 warnings
     openh264                  : NO Program 'nasm' not found or not executable
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
