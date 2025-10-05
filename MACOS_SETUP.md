# macOS M1 Setup Status for Open Remote Play

## Summary

The open-rp project was successfully cloned and configured, but compilation fails due to incompatibilities with modern macOS libraries.

## What Was Completed

1. ✅ Cloned repository with submodules
2. ✅ Installed dependencies via Homebrew:
   - sdl12-compat (SDL 1.2 compatibility layer)
   - sdl2_ttf, sdl2_image, sdl2_net
   - freetype, openssl, curl, ffmpeg
   - autoconf, automake, libtool, pkg-config
   - wxwidgets
3. ✅ Generated build system (autogen.sh)
4. ✅ Modified configure.ac to use SDL2 libraries
5. ✅ Successfully ran ./configure
6. ⚠️  Build fails with multiple compatibility errors

## Compatibility Issues

### 1. SDL 1.2 vs SDL2 API Incompatibility
- The code uses SDL 1.2 headers (`<SDL/SDL.h>`)
- macOS only provides SDL2 libraries
- SDL2 headers use macros like `SDL_FORCE_INLINE` not defined in SDL 1.2
- Created symlinks for SDL extension libraries, but API differences prevent compilation

### 2. Deprecated FFmpeg API
The code uses old FFmpeg API functions that have been removed:
- `avcodec_open()` → replaced with `avcodec_open2()`
- `avcodec_alloc_frame()` → replaced with `av_frame_alloc()`
- `PIX_FMT_YUV420P` → renamed to `AV_PIX_FMT_YUV420P`

## Current Build Errors

```
/opt/homebrew/include/SDL/SDL_net.h:786:1: error: unknown type name 'SDL_FORCE_INLINE'
orp.cpp:556:6: error: use of undeclared identifier 'avcodec_open'
orp.cpp:563:10: error: use of undeclared identifier 'avcodec_alloc_frame'
orp.cpp:571:3: error: use of undeclared identifier 'PIX_FMT_YUV420P'
```

## Next Steps to Fix

To make this project build on modern macOS M1, you would need to:

1. **Port SDL 1.2 code to SDL2**:
   - Update all SDL API calls to SDL2 equivalents
   - This is substantial work as SDL2 has significant API changes

2. **Update FFmpeg API usage**:
   - Replace deprecated function calls with modern equivalents
   - Update pixel format constants

3. **Alternative**: Use Docker/VM with older libraries
   - Run an older Linux distribution with SDL 1.2 and older FFmpeg
   - This avoids porting work but requires container/VM setup

## Files Modified

- `/Users/fource/bytecats/open-rp/configure.ac` - Changed SDL_ttf/image/net to SDL2_ttf/image/net
- `/Users/fource/bytecats/open-rp/m4/wxwin.m4` - Added wxWidgets macros
- `/opt/homebrew/include/SDL/` - Created symlinks for SDL2 extension headers

## Recommendation

This project appears unmaintained (last updated several years ago) and targets PSP/PS3 Remote Play which Sony has since deprecated. Consider looking for:
- Alternative modern remote play solutions
- A fork of this project that's been updated for modern libraries
- Running this in a Linux VM with older library versions
