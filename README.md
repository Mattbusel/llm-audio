# llm-audio

[![CI](https://github.com/Mattbusel/llm-audio/actions/workflows/ci.yml/badge.svg)](https://github.com/Mattbusel/llm-audio/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)
![C++17](https://img.shields.io/badge/C%2B%2B-17-blue.svg)
![Single header](https://img.shields.io/badge/single-header-green.svg)

Speech-to-text, audio translation and text-to-speech for C++ through the OpenAI audio API, in one header.

> Part of **[llm-cpp](https://github.com/Mattbusel/llm-cpp)**, a family of 26 single-header C++ libraries for building on LLM APIs. Each one stands alone: copy one header, include it, done.

Adding voice to a C++ app usually means pulling in an SDK or hand-writing multipart uploads. llm-audio wraps OpenAI's Whisper transcription, translation and TTS endpoints in a handful of functions that take a file path (or bytes) and return text (or audio).

## Features

- Transcribe audio files (mp3, mp4, wav, m4a, ogg, webm) or raw bytes with Whisper
- Translate speech in any supported language into English text
- Output formats: `text`, `json`, `srt`, `vtt`; optional language hint, context prompt and temperature
- Text-to-speech with six voices (Alloy, Echo, Fable, Onyx, Nova, Shimmer), MP3/Opus/AAC/FLAC output and speed control
- TTS straight to a file or to a `std::vector<uint8_t>`

## Quick start

Requirements: a C++17 compiler and libcurl (`apt install libcurl4-openssl-dev`, `brew install curl`, or `vcpkg install curl`).

1. Copy [`include/llm_audio.hpp`](include/llm_audio.hpp) into your project.
2. In exactly one `.cpp` file, `#define LLM_AUDIO_IMPLEMENTATION` before including it. Other files just `#include "llm_audio.hpp"`.

```cpp
#define LLM_AUDIO_IMPLEMENTATION
#include "llm_audio.hpp"
#include <cstdlib>
#include <iostream>

int main() {
    const char* key = std::getenv("OPENAI_API_KEY");

    // Speech to text (Whisper)
    llm::TranscribeConfig stt;
    stt.api_key = key;
    llm::TranscribeResult r = llm::transcribe("meeting.mp3", stt);
    std::cout << r.text << "\n";

    // Text to speech
    llm::TTSConfig tts;
    tts.api_key = key;
    tts.voice   = llm::TTSVoice::Nova;
    llm::text_to_speech("Build finished. All tests passed.", "done.mp3", tts);
}
```

Build and run:

```bash
g++ -std=c++17 -I include example.cpp -o example -lcurl
export OPENAI_API_KEY=sk-...
./example
```

## API

Everything lives in namespace `llm`.

| Function / type | What it does |
|---|---|
| `transcribe(path, cfg)` / `transcribe_bytes(bytes, filename, cfg)` | Speech to text via `/v1/audio/transcriptions` |
| `translate(path, cfg)` / `translate_bytes(bytes, filename, cfg)` | Speech in any language to English text via `/v1/audio/translations` |
| `text_to_speech(text, out_path, cfg)` / `text_to_speech_bytes(text, cfg)` | Text to audio via `/v1/audio/speech` |
| `TranscribeConfig`, `TranslateConfig`, `TTSConfig` | API key, model (`whisper-1`, `tts-1`), format, voice, speed, timeout |

## How it works

Transcription and translation build a multipart form upload with libcurl and return a `TranscribeResult` (text, language, duration, format). TTS posts a JSON body and streams the returned audio bytes to memory or disk. Errors are thrown as `std::runtime_error`.

## Examples

The [`examples/`](examples) folder has runnable programs:

- [`pipeline.cpp`](examples/pipeline.cpp)
- [`text_to_speech.cpp`](examples/text_to_speech.cpp)
- [`transcribe.cpp`](examples/transcribe.cpp)
- [`transcribe_file.cpp`](examples/transcribe_file.cpp)
- [`transcribe_url.cpp`](examples/transcribe_url.cpp)
- [`translate.cpp`](examples/translate.cpp)
- [`tts.cpp`](examples/tts.cpp)
- [`tts_voices.cpp`](examples/tts_voices.cpp)

Build the examples with CMake (needs libcurl):

```bash
cmake -B build
cmake --build build
```

Examples that call the API read `OPENAI_API_KEY` from the environment.

## Limitations

- OpenAI endpoints only (URLs are fixed in the header).
- Requests are synchronous, with no retry logic.

## License

MIT. See [LICENSE](LICENSE).
