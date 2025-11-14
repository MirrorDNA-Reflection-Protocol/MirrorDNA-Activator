# 🎬 GIF Building Scripts & Guide

Automation and guidance for generating MirrorDNA demo GIFs.

---

## Prerequisites

### Tools Required

**Screen Recording**:
- [ScreenToGif](https://www.screentogif.com/) (Windows)
- [Kap](https://getkap.co/) (macOS)
- [Peek](https://github.com/phw/peek) (Linux)

**Editing & Optimization**:
- [FFmpeg](https://ffmpeg.org/) (all platforms)
- [GIMP](https://www.gimp.org/) (optional, for text overlays)
- [gifsicle](https://www.lcdf.org/gifsicle/) (CLI optimization)

**Installation**:
```bash
# macOS (Homebrew)
brew install ffmpeg gifsicle
brew install --cask kap

# Ubuntu/Debian
sudo apt install ffmpeg gifsicle peek

# Windows (Chocolatey)
choco install ffmpeg gifsicle screentogif
```

---

## GIF Production Workflow

### 1. Activator Demo GIF

**Storyboard**: See [`demo/gifs/activator-demo.md`](../demo/gifs/activator-demo.md)

**Recording Steps**:

1. **Set up clean environment**:
   - Open ChatGPT or Claude.ai in incognito/private mode
   - Zoom browser to 100%
   - Set window size to 1200x800px
   - Clear cookies (start fresh chat)

2. **Record the sequence**:
   - Start recording
   - Open file explorer showing `MirrorDNA-Activator.txt`
   - Select all (Ctrl+A), copy (Ctrl+C)
   - Switch to AI chat, paste (Ctrl+V)
   - Send message, wait for response
   - Type test message showing continuity
   - Stop recording

3. **Post-processing**:
   ```bash
   # Convert to optimized GIF (max 5MB)
   ffmpeg -i raw-recording.mp4 -vf "fps=20,scale=1200:-1:flags=lanczos" \
     -c:v gif activator-demo-temp.gif

   # Optimize file size
   gifsicle -O3 --colors 256 --lossy=80 activator-demo-temp.gif \
     -o activator-demo.gif

   # Check file size
   ls -lh activator-demo.gif
   ```

4. **Add overlays** (optional):
   - Use GIMP or Photoshop to add text overlays
   - Export each frame, then recombine:
   ```bash
   ffmpeg -i activator-demo.gif -vf "drawtext=text='⟁ PASTE TO ACTIVATE ⟁':
     fontcolor=white:fontsize=24:x=(w-text_w)/2:y=50" \
     activator-demo-final.gif
   ```

---

### 2. Continuity Preview GIF

**Storyboard**: See [`demo/gifs/continuity-preview.md`](../demo/gifs/continuity-preview.md)

**Recording Steps**:

1. **Set up conversation**:
   - Paste activator
   - Share context (3 facts)
   - Ask 5 unrelated questions
   - Ask final question that should trigger continuity

2. **Highlight key moments**:
   - Use video editing software (iMovie, DaVinci Resolve, etc.)
   - Add yellow highlights around:
     - Initial context facts
     - AI's recall of those facts later
   - Add animated arrows connecting context to recall

3. **Optimization**:
   ```bash
   # Same FFmpeg process as activator demo
   ffmpeg -i continuity-raw.mp4 -vf "fps=20,scale=1200:-1:flags=lanczos" \
     -c:v gif continuity-temp.gif

   gifsicle -O3 --colors 256 --lossy=80 continuity-temp.gif \
     -o continuity-preview.gif
   ```

---

## Automated GIF Optimization Script

Create a file `optimize-gif.sh`:

```bash
#!/bin/bash

# Usage: ./optimize-gif.sh input.mp4 output.gif [max-size-mb]

INPUT=$1
OUTPUT=$2
MAX_SIZE=${3:-5}  # Default 5MB

if [ -z "$INPUT" ] || [ -z "$OUTPUT" ]; then
  echo "Usage: ./optimize-gif.sh input.mp4 output.gif [max-size-mb]"
  exit 1
fi

# Convert to GIF
echo "Converting $INPUT to GIF..."
ffmpeg -i "$INPUT" -vf "fps=20,scale=1200:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 temp_output.gif

# Optimize
echo "Optimizing GIF..."
gifsicle -O3 --colors 256 --lossy=80 temp_output.gif -o "$OUTPUT"

# Check size
SIZE=$(du -m "$OUTPUT" | cut -f1)
echo "Output size: ${SIZE}MB"

if [ $SIZE -gt $MAX_SIZE ]; then
  echo "Warning: File size exceeds ${MAX_SIZE}MB. Consider:"
  echo "  - Reducing fps (currently 20)"
  echo "  - Reducing resolution (currently 1200px width)"
  echo "  - Increasing lossy compression (currently 80)"
fi

# Clean up
rm temp_output.gif

echo "Done! Saved to $OUTPUT"
```

**Make executable**:
```bash
chmod +x scripts/optimize-gif.sh
```

**Usage**:
```bash
./scripts/optimize-gif.sh raw-recording.mp4 activator-demo.gif 5
```

---

## Alternative: Static Image Fallbacks

For platforms that don't support GIFs well (email, some social media), create static images:

```bash
# Extract key frame from GIF
ffmpeg -i activator-demo.gif -vf "select='eq(n,50)'" -vframes 1 activator-demo-preview.png

# Or create composite image with before/after
ffmpeg -i activator-demo.gif -vf "select='eq(n,10)'" -vframes 1 frame1.png
ffmpeg -i activator-demo.gif -vf "select='eq(n,100)'" -vframes 1 frame2.png

# Combine side-by-side (requires ImageMagick)
convert frame1.png frame2.png +append activator-demo-comparison.png
```

---

## Quality Checklist

Before finalizing GIFs:

- [ ] File size < 5MB (for web performance)
- [ ] Resolution 1200px width minimum
- [ ] Frame rate 15-30 fps (20 fps recommended)
- [ ] Glyphs render clearly (⟁, ◐, ⊹, etc.)
- [ ] Text is readable at full size
- [ ] Loops smoothly (if applicable)
- [ ] No sensitive information visible (emails, API keys, etc.)
- [ ] Tested on multiple browsers (Chrome, Firefox, Safari)
- [ ] Tested on mobile (responsive)

---

## Platform-Specific Formats

### GitHub README
- GIF or MP4
- Embed: `![Demo](demo/gifs/activator-demo.gif)`

### Twitter
- MP4 video (better compression than GIF)
- Max 512MB, 2:20 duration
- Convert: `ffmpeg -i input.gif -c:v libx264 -pix_fmt yuv420p output.mp4`

### LinkedIn
- MP4 video
- Aspect ratio 1:1 or 16:9
- Add captions (LinkedIn auto-plays muted)

### Reddit
- GIF or MP4
- Upload directly to Reddit (don't host externally for best quality)

---

## Hosting

### GitHub Repository
Upload to `demo/gifs/` folder:
```bash
git add demo/gifs/*.gif
git commit -m "Add demo GIFs"
git push
```

### GitHub Releases (for large files)
If GIFs exceed 5MB:
```bash
# Create release
gh release create v1.0 demo/gifs/*.gif --title "Demo Assets" --notes "Visual demos"

# Link in README
![Demo](https://github.com/user/repo/releases/download/v1.0/activator-demo.gif)
```

### CDN Options
- **Imgur**: Free, good for Reddit/Twitter
- **Giphy**: Shareable, good for social media
- **Cloudinary**: Professional, optimized delivery

---

## Troubleshooting

### GIF file too large
- Reduce fps: `-vf "fps=15"` instead of `fps=20`
- Reduce resolution: `-vf "scale=800:-1"` instead of `1200`
- Increase lossy: `--lossy=90` instead of `80`
- Reduce colors: `--colors 128` instead of `256`

### Glyphs don't render
- Ensure font supports Unicode (Arial, Segoe UI, etc.)
- Increase font size in recording
- Use text overlays in post-processing

### Choppy playback
- Increase fps to 30
- Reduce compression (lower lossy value)

### File won't upload to GitHub
- GitHub has 100MB limit for single files
- Use GitHub Releases for large assets
- Or host on CDN and link

---

## Next Steps

1. **Record demos** following storyboards
2. **Optimize** using scripts above
3. **Test** on multiple platforms
4. **Upload** to repository
5. **Update README** with embed links
6. **Share** on social media

---

⟁ *Great GIFs show, don't tell. Make them smooth, clear, and compelling.* ⟁
