# Runtime Camera to MP4 Exporter

Runtime Camera to MP4 Exporter is a lightweight runtime exporter for Camera-to-MP4 and PNG sequence capture, with optional deterministic frame control.

MP4 files are encoded with Windows Media Foundation. No external executable, installer, command-line tool, or separately downloaded encoder is required.

## Compatibility

- Tested with Unity 2022.3 LTS and Unity 6000.4.
- Native MP4 export is available on Windows only.
- Even width and height values for H.264 MP4 output.

The exporter does not depend on a specific render pipeline. Ready-to-use sample scenes are supplied for Built-in, HDRP, and URP.

## Installation

Import the asset from `My Assets` using Unity Package Manager. All source code is included under `Assets/Runtime Camera to MP4 Exporter`.

## Included scenes

The asset includes two ready-to-use Built-in Render Pipeline scenes under `Runtime Camera to MP4 Exporter/Samples/Exporter Scenes`:

- `VisualEffectsScene`: animated objects, lights, and particle prefabs using normal Unity time-based animation.
- `MidiPhysicsScene`: loads MIDI Note On events and uses `OnFrameRequested` for deterministic frame-by-frame physics.

Matching HDRP versions and their HDRP materials are provided in the optional `Runtime Camera to MP4 Exporter/Samples/HDRP Samples.unitypackage`:

- `VisualEffectsScene_HDRP`
- `MidiPhysicsScene_HDRP`

To install them, first install and configure HDRP in the destination project, then double-click `HDRP Samples.unitypackage` and import its contents. Do not import this optional sample package into a Built-in or URP project.

Matching URP versions and their URP materials are provided in the optional `Runtime Camera to MP4 Exporter/Samples/URP Samples.unitypackage`:

- `VisualEffectsScene_URP`
- `MidiPhysicsScene_URP`

To install them, first install and configure URP in the destination project, then double-click `URP Samples.unitypackage` and import its contents. Do not import this optional sample package into a Built-in or HDRP project. These URP samples were validated with Unity 2022.3 LTS and Unity 6000.4.

The scene-switch buttons work directly in the Unity Editor even when the scenes have not been added to the project Build Settings. To use scene switching in a standalone demo build, add both scenes from the installed sample set (Built-in, HDRP, or URP) to `File > Build Settings` (Unity 2022.3) or the active Build Profile (Unity 6).

## Quick setup

1. Select the Camera that should be exported.
2. Add the `Runtime Camera Mp4 Exporter` component.
3. Assign `Source Camera` if it was not assigned automatically.
4. Select an export preset or configure the resolution, FPS, frame count, bitrate, and output path.
5. Enter Play Mode.
6. Click `Start MP4 Export` in the Inspector, or call `StartExport()` from your own runtime UI or code.

The default configured relative output path is `Exports/export.mp4`. Because `New Folder Per Render` is enabled by default, the resulting file is normally written as `Exports/Render_<timestamp>/export.mp4`.

- In the Editor, relative paths are resolved from the Unity project directory.
- In a build, relative paths are resolved from `Application.persistentDataPath`.
- Absolute paths are also supported.

`OutputPath` always represents the output location and base MP4 file name. For PNG modes, PNG files are written beside that MP4 path using `PngPrefix`, for example `frame_000000.png`. When `New Folder Per Render` is enabled, the MP4, PNG files, and export report are grouped inside the same timestamped render folder.

## Export modes

- `Native MP4 Only`: writes an H.264 MP4 directly through Windows Media Foundation.
- `PNG Sequence Only`: writes lossless PNG frames without creating an MP4.
- `Native MP4 Only (Legacy Capture)`: uses the former ARGB32 readback path and, in Game View mode, captures at the current screen resolution before resizing. It is intended for compatibility comparisons.
- `PNG Sequence And MP4 Keep`: writes the PNG sequence and MP4 during the same render pass.

## Capture modes

### Direct Camera

Renders the selected Camera to an offscreen RenderTexture at the requested export resolution. This is usually the sharpest and most predictable mode.

`Display Camera During Export` keeps a preview of the exported Camera visible on screen. Disable it when the Camera only needs to feed the export RenderTexture. The option applies only to Direct Camera capture.

When Direct Camera is used with HDRP in Unity 2022.3, the Editor Game View can temporarily display Unity's `Display 1 - No cameras rendering` message. The exporter is intentionally routing the Camera to its offscreen export target and drawing a preview of that target. This Editor-only message is not included in exported frames, does not indicate a capture failure, and does not appear in a standalone build.

The exporter always restores the Camera target that was assigned before the export. This also happens after cancellation or a handled export error. If the Camera already used a custom RenderTexture, that same RenderTexture is restored automatically.

`Screen Space - Overlay` UI is not rendered by a Camera and is therefore not captured in this mode. Use Camera Space UI or Game View capture when UI must be visible.

### Game View

Captures the final Game View or player frame. Enable `Resize Game View To Export` to request the target resolution during export and restore the previous resolution afterward.

## Unity capture framerate

`Use Unity Capture Framerate` temporarily sets `Time.captureFramerate` to the selected export FPS and restores the previous value when the export ends.

When enabled, Unity advances game time by exactly `1 / FPS` for every rendered frame. `Update`, `Time.deltaTime`, Animator, Particle System, and normal Unity time-based animation therefore advance without skipped frames even when encoding is slower than real time. For example, a 60 FPS export advances by exactly 1/60 second per captured frame.

This is recommended for offline rendering and reproducible exports. Disable it only when the exported video must follow real elapsed time. External clocks, network data, audio devices, and custom code based on wall-clock time are not made deterministic by this option; use `OnFrameRequested` when content must be evaluated at the exact requested frame time.

## Quality and bitrate

Native MP4 output uses H.264 for broad compatibility. Three rate-control modes are available:

- `VBR - Target Bitrate` adapts the bitrate to the content while targeting the selected average bitrate. This is the recommended general-purpose mode.
- `VBR - Constant Quality` adapts the bitrate to maintain the selected quality level from 1 to 100.
- `Constant Bitrate (CBR)` asks the selected Media Foundation encoder to target a stable bitrate.

Media Foundation does not guarantee that every hardware or software encoder will produce the exact requested bitrate. The encoder can limit or underuse the target according to its capabilities and the video content. CBR is therefore a requested target, not a promise of an exact padded output rate.

Hardware encoders can expose different capabilities and can produce a different effective bitrate from the Windows software encoder. If a selected mode is rejected or gives an unsuitable result, disable `Use Hardware Encoding` and compare the output.

Suggested starting points:

| Output | Suggested bitrate |
| --- | ---: |
| 1080p30 | 20-30 Mbps |
| 1080p60 | 35-50 Mbps |
| 4K30 | 60-100 Mbps |
| 4K60 | 100-150 Mbps |

`Use Hardware Encoding` asks Media Foundation to use available hardware transforms. Windows can still choose an available compatible encoder when a hardware path is unavailable.

For lossless image output, use `PNG Sequence Only`.

## Color and chroma

H.264 output uses the NV12 4:2:0 pixel format required by the Media Foundation encoding path. This means one chroma sample is shared by each 2x2 pixel block, regardless of the selected conversion mode.

- `Standard Color` averages the four source pixels when calculating chroma. It is recommended for natural images, gradients, and general 3D rendering.
- `Sharp Color` uses the most saturated source pixel in each 2x2 block when calculating chroma. It can preserve the appearance of small, strongly colored graphics and pixel-art edges, but may exaggerate color on some content.

Neither mode converts H.264 into 4:4:4 or lossless video. Use `PNG Sequence Only` when exact per-pixel color preservation is required.

## Audio capture and export FPS

Enable `Capture Audio` to capture Unity's final audio mix and include it in MP4 output. The captured mix is the sound heard through the active `AudioListener`, including Unity `AudioSource` spatialization and mixer processing.

Audio capture follows the Unity simulation used by the export. Sounds started by `Update`, `OnFrameRequested`, animation events, or other frame-driven scene logic can therefore depend on the selected export FPS. At 1 FPS, for example, that logic is evaluated only once per exported second, so short or rapidly triggered sounds may start late, overlap differently, or be missed by scene code. This is expected behavior for frame-driven events rather than an audio encoding problem.

Use a normal delivery frame rate such as 30 or 60 FPS when capturing general scene audio. The MIDI sample demonstrates a different use case: audio generated from a continuous sample-based timeline, providing consistent timing independently of the export FPS. This timeline-based behavior is implemented by the sample and is not applied automatically to arbitrary `AudioSource` components.

## Deterministic frame control

The exporter invokes `OnFrameRequested(frameIndex, time)` immediately before capturing each frame. Use it when animation, MIDI playback, Timeline, replay data, or simulation must be evaluated at the exact exported time.

```csharp
using AIGLE25.RuntimeCameraMp4Exporter;
using UnityEngine;

public sealed class DeterministicExportExample : MonoBehaviour
{
    [SerializeField] private RuntimeCameraMp4Exporter exporter;
    [SerializeField] private Transform animatedObject;

    private void OnEnable()
    {
        exporter.OnFrameRequested += EvaluateFrame;
    }

    private void OnDisable()
    {
        exporter.OnFrameRequested -= EvaluateFrame;
    }

    private void EvaluateFrame(int frameIndex, double time)
    {
        animatedObject.localPosition = new Vector3(Mathf.Sin((float)time), 0f, 0f);
    }
}
```

For content with a known duration, calculate the frame count automatically:

```csharp
exporter.Settings.FrameCount = Mathf.CeilToInt(durationSeconds * exporter.Settings.FrameRate);
```

## Runtime API example

```csharp
using AIGLE25.RuntimeCameraMp4Exporter;
using UnityEngine;

public sealed class StartCameraExport : MonoBehaviour
{
    [SerializeField] private RuntimeCameraMp4Exporter exporter;

    public void Export()
    {
        exporter.Settings.Width = 1920;
        exporter.Settings.Height = 1080;
        exporter.Settings.FrameRate = 60;
        exporter.Settings.FrameCount = 600;
        exporter.Settings.RateControlMode = RuntimeRenderRateControlMode.VariableBitrate;
        exporter.Settings.VideoBitrateMbps = 50;
        exporter.Settings.UseHardwareEncoding = true;
        exporter.Settings.OutputPath = "Exports/video.mp4";
        exporter.StartExport();
    }
}
```

## Size estimate and output folder

`EstimatedMp4SizeMegabytes` calculates a theoretical MP4 size from the configured target bitrate for CBR and target-bitrate VBR exports. The actual file can be smaller because Media Foundation encoders do not always consume the full requested bitrate. Treat this value as a planning estimate rather than a guaranteed final size. Quality-based VBR and PNG sizes depend entirely on the rendered content, so `HasPredictableMp4Size` returns `false` for those cases.

`Open Folder When Complete` is enabled by default and automatically opens the destination folder after a successful export. It does not open the folder after a cancellation or failure. Disable `OpenOutputFolderOnComplete` if the application should stay in the foreground.

After a successful export, `HasCompletedOutput` and `LastCompletedPath` identify the latest result. Call `OpenOutputFolder()` to open its containing folder manually at any time.

The output folder also receives an `export_settings.txt` report containing the effective resolution, FPS, frame count, encoder settings, output paths, elapsed time, and final status. This report is useful when comparing test renders or diagnosing an encoder problem.

### Output naming

`Custom Name` keeps the file name configured in `OutputPath`. `Date And Time` appends one timestamp when the export starts:

```text
export_2026-08-08_18-35-42-123.mp4
```

The configured name remains available as a prefix. For example, `Gameplay.mp4` becomes `Gameplay_2026-08-08_18-35-42-123.mp4`. `New Folder Per Render` uses the same timestamp for its render folder.

```csharp
exporter.Settings.OutputPath = "Exports/Gameplay.mp4";
exporter.Settings.OutputNamingMode = RuntimeRenderOutputNamingMode.DateAndTime;
```

```csharp
if (exporter.Settings.HasPredictableMp4Size)
    Debug.Log($"Estimated MP4 size: {exporter.Settings.EstimatedMp4SizeMegabytes:0} MB");

if (exporter.HasCompletedOutput)
    exporter.OpenOutputFolder();
```

## Events

- `OnFrameRequested(int frameIndex, double time)`: evaluate custom content before capture.
- `OnProgress(Mp4ExportProgress progress)`: update progress UI and estimated remaining time.
- `OnCompleted(string path)`: receives the completed MP4 file or PNG directory.
- `OnFailed(string error)`: receives cancellation, validation, capture, and encoder errors.

Call `CancelExport()` to stop an active export.

`IsExporting` reports whether an export is active. `LastExportElapsedSeconds` contains the elapsed wall-clock time of the latest completed, cancelled, or failed export. The remaining time supplied through `OnProgress` is an estimate based on completed frames; it can fluctuate near the beginning or when scene complexity changes significantly.

## Current limitations

- MP4 uses H.264 and is not lossless.
- Audio capture is optional and follows Unity's active audio mix and simulation timing.
- Alpha is not preserved in H.264 MP4 output.
- XR, VR, and stereo capture are not officially validated.
- Screen Space Overlay UI is not captured in Direct Camera mode.
- The current capture path reads frames back to CPU memory before encoding.

## Troubleshooting

### Native H.264 export requires even dimensions

Use even width and height values such as 1920x1080 or 1280x720.

### MP4 initialization fails

Confirm that the project is running in Windows Editor or a Windows standalone build. Try disabling `Use Hardware Encoding` to let Media Foundation select another compatible encoder path.

### The MP4 orientation or colors differ from the Game View

First compare `Direct Camera` and `Game View` capture at the same resolution. Direct Camera renders an offscreen Camera target, while Game View captures the final presented frame.

### HDRP 2022.3 displays "No cameras rendering" during Direct Camera export

This is an Editor Game View indicator, not an exporter error. Direct Camera temporarily routes the Camera to the offscreen export target so HDRP renders each frame only once. The message is not captured in the video and does not appear in a standalone build.

### My animation plays in real time instead of exact exported frames

Subscribe to `OnFrameRequested` and manually evaluate the scene using the provided `time` value.
