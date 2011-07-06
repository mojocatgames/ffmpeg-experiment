#Note

This proof of concept has only been tested on my Acer 3810T running Ubuntu 11.04.

It uses ffmpeg settings that are slower, but produce higher quality. 

Much credit owed to FakeOutdoorsman from [this thread](http://ubuntuforums.org/showthread.php?t=786095).

#Installation

###1. Uninstall existing ffmpeg and libraries, then download the latest. We're starting clean.

```sh
sudo apt-get remove ffmpeg x264 libx264-dev libvpx-dev
sudo apt-get update
sudo apt-get install build-essential git-core checkinstall yasm texi2html \
    libfaac-dev libjack-jackd2-dev libmp3lame-dev libopencore-amrnb-dev \
    libopencore-amrwb-dev libsdl1.2-dev libtheora-dev libva-dev libvdpau-dev \
    libvorbis-dev libx11-dev libxfixes-dev libxvidcore-dev zlib1g-dev
```

###2. Grab a fresh copy of our test video, rename it video.avi

@TODO get download url

```sh
cd path_to_cloned_repo/input
wget ......
mv name_of_video.avi video.avi
```

###3. Install transcode, x264, libvpx (someday we might want to support webM), and ffmpeg (with libavfilter).

```sh
cd path_to_cloned_repo/lib

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

# install transcode
sudo apt-get install transcode
```

###4. Run the script!

```
cd path_to_cloned_repo/
./convert_videos
```
