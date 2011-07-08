#Note

This proof of concept has only been tested on my Acer 3810T running Ubuntu 11.04. Most likely works on Mac.

FFmpeg settings tweaked for lower file sizes at the expense of lower quality and greater encoding time.

Much credit owed to FakeOutdoorsman from [this thread](http://ubuntuforums.org/showthread.php?t=786095).

#Installation

###0. If you're on Windows, install Ubuntu.

Visit [this page](http://www.ubuntu.com/download/ubuntu/windows-installer) and click 'Start download'.

Follow the prompts, restart, and choose Ubuntu as your OS when the boot screen appears. Complete the installation and open up gnome-terminal.

###1. Update your system. We're starting from scratch.

Included in this bunch are tools required for development and video/audio processing. 

```bash
sudo apt-get remove ffmpeg x264 libx264-dev libvpx-dev
sudo apt-get update
sudo apt-get install build-essential git-core checkinstall yasm texi2html \
    libfaac-dev libjack-jackd2-dev libmp3lame-dev libopencore-amrnb-dev \
    libopencore-amrwb-dev libsdl1.2-dev libtheora-dev libva-dev libvdpau-dev \
    libvorbis-dev libx11-dev libxfixes-dev libxvidcore-dev zlib1g-dev \ 
    ffmpeg2theora transcode
```

###2. Clone this repo

```bash
git clone git://github.com/ezYZ/ffmpeg-experiment.git
```

###3. Grab a copy of our test video(s)

If you don't have a suitable video, ask Ryan K.

You can store these videos anywhere, but for convenience, I've created the `input/` dir.

###4. Install FFmpeg from source

We'll be installing libx264, libvpx (someday, we might want to support webM), ffmpeg, and qt-faststart.

Navigate to your `ffmpeg-experiment/` dir and run:

```bash
cd lib

# clone sources
git clone git://git.videolan.org/x264
git clone git://review.webmproject.org/libvpx
git clone git://git.videolan.org/ffmpeg

# install libx264
cd x264
./configure --enable-static
make
sudo checkinstall --pkgname=x264 --pkgversion="3:$(./version.sh | \
  awk -F'[" ]' '/POINT/{print $4"+git"$5}')" --backup=no --deldoc=yes \
  --fstrans=no --default

# install libvpx
cd ../libvpx
./configure
make
sudo checkinstall --pkgname=libvpx --pkgversion="1:$(date +%Y%m%d%H%M)-git" --backup=no \
  --deldoc=yes --fstrans=no --default

# install ffmpeg
cd ../ffmpeg
./configure \
  --enable-avfilter --enable-gpl --enable-version3 --enable-nonfree --enable-postproc \
  --enable-libfaac --enable-libmp3lame --enable-libopencore-amrnb --enable-libopencore-amrwb \
  --enable-libx264 --enable-libtheora --enable-libvorbis --enable-libxvid --enable-x11grab
make
sudo checkinstall --pkgname=ffmpeg --pkgversion="5:$(date +%Y%m%d%H%M)-git" --backup=no \
  --deldoc=yes --fstrans=no --default

# install qt-faststart
make tools/qt-faststart
sudo checkinstall --pkgname=qt-faststart --pkgversion="$(date +%Y%m%d%H%M)-git" --backup=no \
  --deldoc=yes --fstrans=no --default install -D -m755 tools/qt-faststart \
  /usr/local/bin/qt-faststart
```

###5. Run the script!

Navigate back to your `ffmpeg-experiment/` dir and run:

```bash
./convert input/your_video_file.avi
```

To remove all temporary and output files, run:

```bash
./reset
```
