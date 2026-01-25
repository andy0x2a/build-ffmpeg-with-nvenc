# Building FFmpeg from Source with NVENC Support

A comprehensive guide for compiling FFmpeg with NVIDIA hardware encoding support on Linux systems.

## Prerequisites

Before starting, ensure you have:
- A Linux system (Ubuntu/Debian-based)
- NVIDIA GPU with encoding capabilities
- CUDA Toolkit installed ([Download here](https://developer.nvidia.com/cuda-downloads?target_os=Linux))
- Root/sudo access

## Step 1: Remove Existing FFmpeg

First, remove any system-installed FFmpeg packages to avoid conflicts:

```bash
sudo apt remove --purge ffmpeg
sudo apt autoremove
```

## Step 2: Install Build Dependencies

Update your package manager and install essential build tools:

```bash
sudo apt update
sudo apt install build-essential git pkg-config
```

## Step 3: Install NVENC Headers

The NVENC headers are required for hardware-accelerated HEVC encoding (`hevc_nvenc`):

```bash
git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git
cd nv-codec-headers
make
sudo make install
cd ..
```

## Step 4: Download and Extract FFmpeg

Download FFmpeg 7.1.1 source code:

```bash
wget https://ffmpeg.org/releases/ffmpeg-7.1.1.tar.xz
tar -xvf ffmpeg-7.1.1.tar.xz
cd ffmpeg-7.1.1/
```

## Step 5: Install Additional Dependencies

Install all required libraries for codec support:

```bash
sudo apt-get update -qq && sudo apt-get -y install \
    autoconf \
    automake \
    build-essential \
    cmake \
    git \
    git-core \
    libass-dev \
    libc6 \
    libc6-dev \
    libdav1d-dev \
    libfdk-aac-dev \
    libfreetype6-dev \
    libgnutls28-dev \
    libmp3lame-dev \
    libnuma-dev \
    libnvidia-encode-570 \
    libopus-dev \
    libsdl2-dev \
    libtool \
    libva-dev \
    libvdpau-dev \
    libvorbis-dev \
    libvpx-dev \
    libx264-dev \
    libx265-dev \
    libxcb-shm0-dev \
    libxcb-xfixes0-dev \
    libxcb1-dev \
    meson \
    ninja-build \
    pkg-config \
    texinfo \
    unzip \
    wget \
    yasm \
    zlib1g-dev
```

## Step 6: Configure FFmpeg

Configure the build with CUDA and NVENC support:

```bash
./configure \
    --prefix=/usr/local \
    --extra-cflags="-I/usr/include" \
    --extra-ldflags="-L/usr/lib/x86_64-linux-gnu" \
    --extra-libs="-lpthread -lm" \
    --ld=g++ \
    --enable-gpl \
    --enable-nonfree \
    --enable-shared \
    --enable-cuda \
    --enable-cuvid \
    --enable-cuda-nvcc \
    --enable-nvenc \
    --enable-libass \
    --enable-libfdk-aac \
    --enable-libfreetype \
    --enable-libmp3lame \
    --enable-libopus \
    --enable-libvorbis \
    --enable-libvpx \
    --enable-libx264 \
    --enable-libx265 \
    --enable-libdav1d \
    --enable-sdl2
```

## Step 7: Compile and Install

Build FFmpeg using all available CPU cores:

```bash
make -j$(nproc)
sudo make install
sudo ldconfig
hash -r
```

## Step 8: Verify Installation

Check that hardware acceleration and NVENC encoders are available:

```bash
# Check available hardware accelerators
ffmpeg -hwaccels

# Verify NVENC encoders are installed
ffmpeg -encoders | grep nvenc
```

You should see output listing NVIDIA encoders like:
- `h264_nvenc`
- `hevc_nvenc`
- `av1_nvenc` (if supported by your GPU)

## Troubleshooting

### NVENC encoders not found
- Ensure CUDA Toolkit is properly installed
- Verify your GPU supports NVENC
- Check that `libnvidia-encode` matches your driver version

### Build failures
- Make sure all dependencies are installed
- Check that you have sufficient disk space
- Review configuration output for missing libraries

## Notes

- This build includes both GPL and non-free components due to `--enable-gpl` and `--enable-nonfree` flags
- The `libfdk-aac` encoder provides high-quality AAC encoding but requires the non-free flag
- Hardware acceleration significantly reduces encoding time and CPU usage

## License

FFmpeg is licensed under the LGPL or GPL depending on the configuration. This build uses GPL due to the included libraries.

---

**Last Updated**: December 2025 
**FFmpeg Version**: 7.1.1  
**Tested On**: Pop!_OS 22.04 a fork of Ubuntu 
