# Vision/Analyze Tool: Silent Image Rejection by Non-Vision Models

## Symptom

`vision_analyze` returns `"success": true` with `"analysis": "I don't see any image attached..."` despite:
- Local file path correctly resolved
- Image file valid (JPEG, PNG, etc.)
- base64 conversion successful
- HTTP 200 OK from API
- No error thrown

## Root Cause Pattern

**The configured model does not support multimodal/vision input.** The API accepts the request format (OpenAI-wire or Anthropic format), returns 200, but the model is text-only and ignores image data entirely.

This commonly occurs with:
- `MiniMax-M2.7` — text-only model, no vision support
- Other pure-text models used as `auxiliary.vision.provider: auto`

## Diagnostic Steps

1. Enable debug logging:
   ```bash
   VISION_TOOLS_DEBUG=1 hermes chat -q "..."
   ```

2. Look for: `"Vision auto-detect: using main provider X (Y)"` — this confirms the model being used.

3. Confirm with direct API test:
   ```python
   # If MiniMax returns "no image attached", vision is not supported
   ```

4. Check `auxiliary.vision.provider` config:
   ```bash
   hermes config show | grep -A10 auxiliary:
   ```

## Fix Options

### Option A: Add a Vision-Capable Provider (Recommended)

Add OpenRouter API key (has many vision models):
```bash
# ~/.hermes/.env
OPENROUTER_API_KEY=sk-or-v1-xxxxx

# config.yaml
auxiliary:
  vision:
    provider: openrouter
```

Restart hermes. OpenRouter will route to `google/gemini-3-flash-preview` or similar.

### Option B: Configure Local Vision Model

If running Ollama with a vision model (Qwen-VL, LLaVA, etc.):
```bash
auxiliary:
  vision:
    provider: custom
    base_url: http://localhost:11434/v1
    model: qwen-vl
```

### Option C: Switch Main Model

Use a vision-capable model as the primary model:
```bash
hermes model
# Select a model with vision support (Gemini Pro, GPT-4o, etc.)
```

## Key Diagnostic Log Lines

| Log Line | Meaning |
|----------|---------|
| `Vision auto-detect: using main provider minimax-cn (MiniMax-M2.7)` | Using non-vision model |
| `Image ready (1365.2 KB)` | Local file correctly found |
| `Image converted to base64 (1820.3 KB)` | Image encoding works |
| `HTTP Request: POST .../v1/chat/completions "HTTP/1.1 200 OK"` | API call succeeded |
| `"analysis": "no image attached"` | Model silently rejected image |

## MiniMax Specifics

- Endpoint: `https://api.minimaxi.com/anthropic` → rewritten to `/v1`
- Model for aux tasks: `MiniMax-M2.7` (text-only)
- `_convert_openai_images_to_anthropic()` correctly converts format
- But model itself ignores image blocks
- No error thrown — model returns 200 with text-only response

## Prevention

Before relying on `vision_analyze`, verify your `auxiliary.vision` config:
```bash
hermes config show | grep -A5 auxiliary.vision
```

If `provider: auto` and main model is text-only, vision will silently fail.
