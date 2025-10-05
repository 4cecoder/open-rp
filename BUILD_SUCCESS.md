# Open Remote Play - macOS M1 Build Success

## ✅ Build Status: SUCCESSFUL

The project has been successfully built for macOS M1 (Apple Silicon)!

## Built Binaries

- **orp** (966K) - Main Open Remote Play executable
- **keys/keys** (52K) - Key management utility
- **util/config-debug** (39K) - Configuration debug utility

## Changes Made

### 1. SDL Compatibility
- Added `SDL_FORCE_INLINE` macro definition for SDL2 header compatibility
- Created symlinks for SDL2 extension headers (SDL_image.h, SDL_net.h, SDL_ttf.h)
- Added `#undef main` for macOS to prevent SDL main override issues

### 2. FFmpeg API Modernization
Updated deprecated FFmpeg functions to modern equivalents:
- `avcodec_alloc_frame()` → `av_frame_alloc()`
- `av_free(frame)` → `av_frame_free(&frame)`
- `avcodec_open()` → `avcodec_open2()`
- `avcodec_close()` → `avcodec_free_context()`
- `avcodec_decode_video2()` → `avcodec_send_packet()` + `avcodec_receive_frame()`
- `avcodec_decode_audio3()` → `avcodec_send_packet()` + `avcodec_receive_frame()`
- `avcodec_register_all()` → Removed (auto-registration in FFmpeg 4.0+)
- `PIX_FMT_YUV420P` → `AV_PIX_FMT_YUV420P`
- `CODEC_ID_*` → `AV_CODEC_ID_*`
- `FF_INPUT_BUFFER_PADDING_SIZE` → `AV_INPUT_BUFFER_PADDING_SIZE`
- `AVCODEC_MAX_AUDIO_FRAME_SIZE` → Hard-coded value (192000 * 3 / 2)
- `AVPicture` → Direct uint8_t arrays
- `AVCodec *` → `const AVCodec *` (FFmpeg 5.0+ requirement)
- `context->channels` → `context->ch_layout.nb_channels`

### 3. Other Fixes
- Fixed `Get_YUV_From_Surface()` to return 0 on success
- Updated `configure.ac` to use SDL2 library names

## Build Warnings

The build completed with 15 warnings (non-critical):
- Deprecated OpenSSL AES functions (AES_cbc_encrypt, AES_set_encrypt_key, etc.)
- Incomplete switch statements for SDL keyboard events
- Incomplete switch statement for CURL info types
- One intentional `#warning` about video sync

These warnings don't affect functionality but could be addressed in future updates.

## How to Build

```bash
# Install dependencies
brew install sdl12-compat sdl2_ttf sdl2_image sdl2_net freetype openssl curl ffmpeg autoconf automake libtool pkg-config wxwidgets

# Create symlinks for SDL extension headers
ln -sf /opt/homebrew/include/SDL2/SDL_image.h /opt/homebrew/include/SDL/SDL_image.h
ln -sf /opt/homebrew/include/SDL2/SDL_net.h /opt/homebrew/include/SDL/SDL_net.h
ln -sf /opt/homebrew/include/SDL2/SDL_ttf.h /opt/homebrew/include/SDL/SDL_ttf.h

# Clone and build
git clone --recursive https://github.com/shoobyban/open-rp
cd open-rp
./autogen.sh
./configure --without-wxorp LDFLAGS="-L/opt/homebrew/lib" CPPFLAGS="-I/opt/homebrew/include"
make
```

## Testing

To test if the build works:
```bash
./orp --version  # (if version flag is supported)
./orp --help     # (if help flag is supported)
```

Note: This project is for PSP/PS3 Remote Play which has been deprecated by Sony. The binaries may require specific hardware or network setup to function properly.

## Files Modified

### Source Code
- `orp.h` - Updated AVCodec pointers to const, added SDL compatibility defines
- `orp.cpp` - Modernized all FFmpeg API calls
- `yuv.cpp` - Added return statement
- `main.cpp` - Added macOS main undef
- `configure.ac` - Updated SDL library names

### Created
- `/opt/homebrew/include/SDL/SDL_image.h` (symlink)
- `/opt/homebrew/include/SDL/SDL_net.h` (symlink)
- `/opt/homebrew/include/SDL/SDL_ttf.h` (symlink)

## System Info

- Platform: macOS (Darwin 25.0.0)
- Architecture: arm64 (Apple Silicon M1)
- Compiler: clang++ (Apple LLVM)
- Build Date: October 5, 2025
