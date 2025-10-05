# Open Remote Play - macOS M1/Apple Silicon Build Guide

This fork contains patches to build Open Remote Play on modern macOS with Apple Silicon (M1/M2/M3).

## ✅ Build Status

Successfully builds and runs on:
- macOS 15.0+ (Sequoia)
- Apple Silicon (arm64)
- wxWidgets 3.3.1
- FFmpeg 8.0
- SDL 1.2 (via sdl12-compat)

## Quick Start

### Prerequisites

Install dependencies via Homebrew:
```bash
brew install sdl12-compat sdl2_ttf sdl2_image sdl2_net freetype openssl curl ffmpeg autoconf automake libtool pkg-config wxwidgets
```

### Build Steps

1. **Clone with submodules:**
```bash
git clone --recursive https://github.com/4cecoder/open-rp
cd open-rp
```

2. **Create SDL header symlinks:**
```bash
ln -sf /opt/homebrew/include/SDL2/SDL_image.h /opt/homebrew/include/SDL/SDL_image.h
ln -sf /opt/homebrew/include/SDL2/SDL_net.h /opt/homebrew/include/SDL/SDL_net.h
ln -sf /opt/homebrew/include/SDL2/SDL_ttf.h /opt/homebrew/include/SDL/SDL_ttf.h
```

3. **Generate build files:**
```bash
./autogen.sh
```

4. **Configure:**
```bash
./configure LDFLAGS="-L/opt/homebrew/lib" CPPFLAGS="-I/opt/homebrew/include"
```

5. **Build:**
```bash
make
```

### Running the Application

**GUI Version (recommended):**
```bash
./wxorp/wxorp
```

**Command-line version:**
```bash
./orp <config-file> <ps3-nickname>
```

## What Changed

### Code Modifications

All changes maintain backward compatibility while enabling modern toolchain support:

#### 1. SDL Compatibility (`orp.h`)
- Added `SDL_FORCE_INLINE` macro for SDL2 headers
- Required for SDL 1.2 compatibility layer on modern systems

#### 2. FFmpeg API Modernization (`orp.cpp`)
Updated to FFmpeg 8.0 API:
- `avcodec_open()` → `avcodec_open2()`
- `avcodec_alloc_frame()` → `av_frame_alloc()`
- `av_free(frame)` → `av_frame_free(&frame)`
- `avcodec_close()` → `avcodec_free_context()`
- `avcodec_decode_video2()` → `avcodec_send_packet()` + `avcodec_receive_frame()`
- `avcodec_decode_audio3()` → `avcodec_send_packet()` + `avcodec_receive_frame()`
- `AVPicture` struct → Direct `uint8_t*` arrays
- `PIX_FMT_*` → `AV_PIX_FMT_*`
- `CODEC_ID_*` → `AV_CODEC_ID_*`
- `FF_INPUT_BUFFER_PADDING_SIZE` → `AV_INPUT_BUFFER_PADDING_SIZE`
- `AVCODEC_MAX_AUDIO_FRAME_SIZE` → `192000 * 3 / 2`
- `context->channels` → `context->ch_layout.nb_channels`
- `AVCodec *` → `const AVCodec *`
- Removed deprecated `avcodec_register_all()` call

#### 3. Platform Fixes
- **main.cpp**: Added macOS to `#undef main` for SDL compatibility
- **wxorp.cpp**: Fixed C++11 string literal concatenation (added space)
- **yuv.cpp**: Added missing return statement
- **configure.ac**: Updated SDL library names (SDL_ttf → SDL2_ttf, etc.)

#### 4. Type Updates (`orp.h`)
Updated structs to use const-qualified codec pointers:
- `orpCodec_t`
- `orpThreadVideoDecode_t`
- `orpThreadAudioDecode_t`

### Build System Changes

**configure.ac:**
- Changed library checks from SDL 1.2 to SDL2 variants
- Maintained backward compatibility with `--without-wxorp` option

## Architecture Notes

### SDL 1.2 → SDL2 Transition

The project was written for SDL 1.2 (deprecated). We use:
- **sdl12-compat**: Provides SDL 1.2 API via SDL2 backend
- **Header symlinks**: SDL2 extension libraries (ttf, image, net) → SDL/ directory
- **Compatibility macro**: `SDL_FORCE_INLINE` bridges API differences

### FFmpeg Version Compatibility

The decode API changed significantly between FFmpeg versions:

**Old API (FFmpeg < 3.0):**
```c
avcodec_decode_video2(context, frame, &got_frame, &packet);
```

**New API (FFmpeg 4.0+):**
```c
avcodec_send_packet(context, &packet);
avcodec_receive_frame(context, frame);
```

Our implementation properly handles EAGAIN and EOF return codes for frame buffering.

## Build Artifacts

After successful build:
- **orp** (966KB) - Command-line Remote Play client
- **wxorp/wxorp** (352KB) - GUI application
- **keys/keys** (52KB) - Key management utility
- **util/config-debug** (39KB) - Configuration debug tool

## Known Issues

### Build Warnings (Non-Critical)
- OpenSSL AES functions deprecated (15 warnings)
- Incomplete switch statements for SDL keys
- Can be ignored - don't affect functionality

### Runtime Limitations
This is a PSP/PS3 Remote Play client. To use it, you need:
1. PlayStation Portable (PSP) with custom firmware
2. PlayStation 3 console
3. Export configuration from PSP (`export.orp` file)
4. Encryption keys (`keys.orp`)

Sony discontinued PS3 Remote Play, so this is primarily a historical/archival project.

## Dependencies

### Required
- **sdl12-compat**: SDL 1.2 compatibility layer
- **sdl2_ttf**: TrueType font support
- **sdl2_image**: Image loading (PNG support)
- **sdl2_net**: Network functionality
- **ffmpeg**: Video/audio codec support (H.264, AAC, ATRAC3)
- **openssl**: Encryption (AES CBC)
- **curl**: HTTP communication
- **freetype**: Font rendering
- **wxwidgets**: GUI framework (optional)

### Build Tools
- **autoconf**: Generate configure script
- **automake**: Generate Makefiles
- **libtool**: Shared library support
- **pkg-config**: Dependency detection

## Contributing

When submitting changes:
1. Test on macOS Apple Silicon
2. Ensure backward compatibility
3. Update this README if adding new dependencies
4. Keep FFmpeg API usage modern (5.0+)

## Original Project

This is a fork of: https://github.com/shoobyban/open-rp

Original author: dsokoloski
Original project: https://code.google.com/p/open-rp/ (archived)

## License

GNU General Public License v2.0 (see LICENSE.md)

---

**Built with ❤️ for macOS Apple Silicon**
