# Runtime Camera to MP4 Exporter

Export a Unity Camera or the Game View at runtime to H.264 MP4, PNG sequences, PNG/JPEG screenshots, and optional synchronized audio.

The original straightforward MP4/PNG workflow remains available. Deterministic rendering, responsive real-time capture, background PNG workers, GPU readback, experimental NV12 conversion, diagnostics, and scripting APIs can be enabled only when needed.

**[Open the complete documentation](DOCUMENTATION.md)**  
Bug reports: [GitHub repository](https://github.com/AIGLE25/Offline-Runtime-Render-Exporter-Unity)

## Main features

- H.264 MP4 through Windows Media Foundation; no external encoder executable required.
- PNG sequences with bounded background compression and configurable workers.
- PNG or JPEG screenshots with asynchronous readback and background saving.
- Optional synchronized audio: AAC in MP4 or PCM16 WAV beside PNG sequences.
- Deterministic offline export with no intentionally discarded output frames.
- Real-time capture that prioritizes gameplay and reports held frames or audio drops.
- Direct Camera and Game View capture modes.
- Configurable resolution, frame rate, duration, bitrate, hardware encoding, and output naming.
- Public runtime API, progress events, cancellation, validation, memory estimates, and completion reports.

## Compatibility

- Tested with Unity 2022.3 LTS and Unity 6000.4.
- MP4 export supports the Windows Editor and Windows standalone builds.
- PNG sequences and screenshots do not require Media Foundation.
- Built-in Render Pipeline, URP, and HDRP are supported.
- H.264 output requires even dimensions; the exporter can adjust odd resolutions automatically.

## Installation

Import the asset from **Window > Package Manager > My Assets**. The main folder is:

`Assets/Runtime Camera to MP4 Exporter`

Add `RuntimeCameraMp4Exporter` to a GameObject, assign `Source Camera`, choose the export settings, and call `StartExport()` from your UI or code.

## Output modes

| Mode | Result | Audio |
| --- | --- | --- |
| Native MP4 Only | H.264 `.mp4` | Optional AAC |
| PNG Sequence Only | Numbered `.png` files | Optional PCM16 `.wav` |
| MP4 + PNG | MP4 and numbered PNG files | Optional AAC; WAV sidecar can be retained |
| Legacy MP4 | Compatibility MP4 path | Optional AAC |

## Quick Inspector setup

1. Add the exporter component and assign a Camera.
2. Select MP4, PNG, or MP4 + PNG.
3. Choose `Direct Camera` or `Game View`.
4. Select `Deterministic / Offline` or `Real-Time / Prioritize Gameplay`.
5. Configure resolution, FPS, duration, output path, and optional audio.
6. Keep advanced settings at their defaults for the first test.
7. Start the export through the sample UI or your own script.

Real-time mode enables Async GPU Readback automatically. Real-time PNG also requires Background PNG Encoding. GPU NV12 is experimental and available only for MP4-only output.

## Quick API example

```csharp
using AIGLE25.RuntimeCameraMp4Exporter;
using UnityEngine;

public sealed class ExportExample : MonoBehaviour
{
    [SerializeField] private RuntimeCameraMp4Exporter exporter;

    public void StartRecording()
    {
        var settings = exporter.Settings;
        settings.ExportMode = RuntimeRenderExportMode.StreamMp4Only;
        settings.CaptureMode = RuntimeRenderCaptureMode.DirectCamera;
        settings.TimingMode = RuntimeRenderTimingMode.DeterministicOffline;
        settings.RecordingResolution = RuntimeRenderResolutionMode.Configured;
        settings.Width = 1920;
        settings.Height = 1080;
        settings.FrameRate = 60;
        settings.LengthMode = RuntimeRenderLengthMode.Duration;
        settings.DurationSeconds = 10f;
        settings.CaptureAudio = true;
        settings.OutputPath = "Exports/video.mp4";
        settings.OutputNamingMode = RuntimeRenderOutputNamingMode.CustomName;
        exporter.StartExport();
    }

    public void CancelRecording()
    {
        exporter.CancelExport();
    }
}
```

## Important behavior

- Deterministic mode waits for bounded queues instead of discarding output frames.
- Real-time mode can hold the previous image when rendering, readback, or workers cannot sustain the target FPS.
- Increasing queue capacity absorbs bursts but does not increase sustained encoder or PNG compression speed.
- Multiple PNG workers can improve throughput but consume additional CPU and RGBA buffer memory.
- Disabling Async GPU Readback in deterministic mode uses blocking `ReadPixels`.
- Only one export or screenshot render phase can run at a time.

For every setting, API member, capture mode, diagnostic field, sample scene, limitation, and troubleshooting procedure, see **[DOCUMENTATION.md](DOCUMENTATION.md)**.
