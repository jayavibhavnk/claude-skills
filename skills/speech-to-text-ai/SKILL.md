---
name: speech-to-text-ai
description: Implement speech-to-text - transcription, speaker diarization, real-time streaming, and voice commands.
metadata:
  priority: 8
  docs:
    - "https://sdk.vercel.ai/docs/ai-sdk-core/speech-to-text"
  pathPatterns:
    - "**/speech/**"
    - "**/audio/**"
    - "**/transcribe/**"
  bashPatterns:
    - '\bwhisper\b'
    - '\bspeech.?to.?text\b'
    - '\btranscri'
  promptSignals:
    phrases:
      - "speech to text"
      - "transcription"
      - "whisper"
    anyOf:
      - "speech"
      - "audio"
      - "transcribe"
      - "voice"
---

## Speech-to-Text AI

### Basic Transcription

```typescript
import { transcribe } from 'ai';

const { audio, timing } = await transcribe({
  model: 'openai/whisper-3',
  audio: audioData,  // File, URL, or base64
});

// Or with AI Gateway
const { audio, timing } = await transcribe({
  model: 'groq/whisper-large-v3',
  audio: audioData,
});
```

### Transcription Options

```typescript
interface TranscriptionOptions {
  model: string;
  audio: string | File | Buffer;
  language?: string;        // e.g., 'en', 'es', 'fr'
  prompt?: string;          // Context to improve accuracy
  temperature?: number;    // 0-1, creativity vs accuracy
  responseFormat?: 'json' | 'text' | 'srt' | 'verbose_json';
}

// Detailed transcription with timestamps
const { audio, segments } = await transcribe({
  model: 'openai/whisper-3',
  audio: audioFile,
  responseFormat: 'verbose_json',
});

// Access segments with timing
for (const segment of segments) {
  console.log(
    `[${segment.start}s - ${segment.end}s]: ${segment.text}`
  );
}
```

### Real-Time Streaming

```typescript
// Browser-based streaming transcription
async function startRealtimeTranscription(
  onTranscript: (text: string, isFinal: boolean) => void
) {
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });

  const mediaRecorder = new MediaRecorder(stream);
  const audioChunks: Blob[] = [];

  mediaRecorder.ondataavailable = async (event) => {
    audioChunks.push(event.data);

    if (audioChunks.length >= 10) {  // Batch every 10 chunks
      const audioBlob = new Blob(audioChunks, { type: 'audio/webm' });
      const arrayBuffer = await audioBlob.arrayBuffer();

      const { audio } = await transcribe({
        model: 'openai/whisper-3',
        audio: Buffer.from(arrayBuffer).toString('base64'),
      });

      onTranscript(audio, false);
      audioChunks.length = 0;
    }
  };

  mediaRecorder.start(100);  // Collect chunks every 100ms

  return () => mediaRecorder.stop();
}
```

### Speaker Diarization

```typescript
// Speaker identification (requires additional service)
interface DiarizedSegment {
  start: number;
  end: number;
  speaker: string;
  text: string;
}

async function transcribeWithDiarization(
  audio: string | File
): Promise<DiarizedSegment[]> {
  // First, get base transcription with timestamps
  const { segments } = await transcribe({
    model: 'openai/whisper-3',
    audio,
    responseFormat: 'verbose_json',
  });

  // Then, use speaker diarization service (e.g., AssemblyAI, Deepgram)
  const diarization = await fetch('https://api.assemblyai.com/v2/upload', {
    method: 'POST',
    headers: {
      'Authorization': process.env.ASSEMBLYAI_API_KEY!,
    },
    body: audio,
  });

  // Combine transcription with speaker info
  const results: DiarizedSegment[] = [];

  for (const segment of segments) {
    const speaker = await getSpeakerAtTime(segment.start, segment.end);
    results.push({
      start: segment.start,
      end: segment.end,
      speaker,
      text: segment.text,
    });
  }

  return results;
}
```

### Voice Commands

```typescript
interface VoiceCommand {
  pattern: RegExp;
  action: (matches: string[]) => void | Promise<void>;
  description: string;
}

const commands: VoiceCommand[] = [
  {
    pattern: /^search for (.+)/i,
    action: ([query]) => performSearch(query),
    description: 'Search for something',
  },
  {
    pattern: /^play (.+)/i,
    action: ([track]) => playMusic(track),
    description: 'Play a song',
  },
  {
    pattern: /^set (?:a )?reminder (.+)/i,
    action: ([reminder]) => createReminder(reminder),
    description: 'Set a reminder',
  },
  {
    pattern: /^(?:open|go to) (.+)/i,
    action: ([page]) => navigateTo(page),
    description: 'Navigate to a page',
  },
];

async function processVoiceCommand(transcript: string): Promise<void> {
  for (const command of commands) {
    const matches = transcript.match(command.pattern);
    if (matches) {
      await command.action(matches.slice(1));
      return;
    }
  }

  console.log('Unknown command:', transcript);
}
```

### Audio Preprocessing

```typescript
// Improve transcription accuracy with preprocessing
async function preprocessAudio(
  audioFile: File,
  options: {
    noiseReduction?: boolean;
    normalize?: boolean;
    trimSilence?: boolean;
  }
): Promise<Buffer> {
  // Use ffmpeg for audio processing
  const ffmpeg = createFFmpeg({ log: true });
  await ffmpeg.load();

  // Write input file
  ffmpeg.FS('writeFile', 'input.webm', await fetchFile(audioFile));

  let command = '-i input.webm';

  if (options.noiseReduction) {
    command += ' -af noisered=noise_profile_file:0.01';
  }

  if (options.normalize) {
    command += ' -af loudnorm=I=-16:TP=-1.5:LRA=11';
  }

  if (options.trimSilence) {
    command += ' -af silenceremove=start_duration=0.5:start_period=-20dB';
  }

  command += ' -ar 16000 -ac 1 output.wav';  // 16kHz mono for Whisper

  await ffmpeg.run(...command.split(' '));

  const output = ffmpeg.FS('readFile', 'output.wav');
  return Buffer.from(output);
}
```

### Use Cases

| Use Case | Best Model | Notes |
|----------|------------|-------|
| Meeting notes | Whisper Large | Enable timestamps |
| Voice commands | Whisper Turbo | Low latency |
| Call center | Whisper Large | + Speaker diarization |
| Podcast | Whisper Large | High accuracy |
| Dictation | Whisper Turbo | Fast, good accuracy |
| Multi-language | Whisper Large | Best cross-language |

### Best Practices

1. **Clean audio** - Reduce noise, normalize volume
2. **Right model** - Turbo for speed, Large for accuracy
3. **Language hint** - Specify language when known
4. **Context prompt** - Add domain-specific terms
5. **Speaker separation** - Use diarization for meetings
6. **Timestamp alignment** - For video sync
7. **Post-processing** - Fix common errors (e.g., "um", "uh")
