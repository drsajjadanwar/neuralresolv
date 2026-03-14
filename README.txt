Empower your visual research with neuralresolv. Harnessing local LLMs, predictive modelling, and neural networks, it delivers exceptional image and video upscaling. Enhance low-resolution media and uncover vital details in your data with precision, all processed securely offline.

================================================================================
                    ARCHITECTURE & PROCESSING WORKFLOW
================================================================================

[USER INPUT]
     |
     v
+------------------------------------------------------------------------------+
| 1. INITIALIZATION & UI THREAD                                                |
|  - User selects file (.png, .jpg, .mp4, .mkv, etc.)                          |
|  - Auto-detect Media Type (Image vs Video) based on extension.               |
|  - FFmpeg invisibly probes native dimensions & auto-fills UI.                |
|  - Aspect Ratio Calculator keeps WxH locked mathematically.                  |
+------------------------------------------------------------------------------+
     |
     v (User clicks "Upscale")
+------------------------------------------------------------------------------+
| 2. ASYNC TASK MANAGER & LIVE TELEMETRY                                       |
|  - UI locked to prevent cross-thread crashes or duplicate task execution.    |
|  - Progress UI & Abort listeners initialized.                                |
|  - [Telemetry Thread]: Polls System CPU, RAM, & GPU (via nvidia-smi) to      |
|    render a real-time hardware performance graph on the dashboard.           |
+------------------------------------------------------------------------------+
     |
     +-----------------------------------------+
     |                                         |
     v [IMAGE PIPELINE]                        v [VIDEO PIPELINE]
+-------------------------+       +--------------------------------------------+
| 3A. IMAGE PROCESSING    |       | 3B. VIDEO PROCESSING (Frame-by-Frame)      |
|                         |       |                                            |
| Mode: Scale by Ratio    |       | Step 1: Extraction                         |
|  -> Real-ESRGAN runs    |       |  -> FFmpeg reads target FPS.               |
|     directly to target. |       |  -> FFmpeg extracts all frames losslessly  |
|                         |       |     into a /temp_in/ directory.            |
| Mode: Exact Resolution  |       |                                            |
|  -> Real-ESRGAN forces  |       | Step 2: AI Batch Upscaling                 |
|     a massive 4x upscale|       |  -> Real-ESRGAN upscales the entire folder |
|     to a temp file.     |       |     to /temp_out/.                         |
|  -> FFmpeg instantly    |       |  -> Async monitor counts generated files   |
|     downscales/stretches|       |     to calculate accurate global ETA.      |
|     the 4x temp file to |       |                                            |
|     the exact WxH target|       | Step 3: GPU Reassembly                     |
|     resolution.         |       |  -> FFmpeg utilizes hardware acceleration  |
|                         |       |     (HEVC_NVENC) to re-encode the frames   |
|                         |       |     and merge the original audio track.    |
|                         |       |  -> (If Exact Resolution is selected, a    |
|                         |       |     scale filter is applied here on-the-fly|
|                         |       |     during hardware encoding).             |
+-------------------------+       +--------------------------------------------+
     |                                         |
     +-----------------------------------------+
     |
     v
+------------------------------------------------------------------------------+
| 4. FINALIZATION & TRUTH CHECK                                                |
|  - Cleanup Phase: Thousands of temporary frames and temp directories deleted.|
|  - Truth Check: Verifies the final file actually exists AND is > 1KB.        |
|    (Prevents FFmpeg from reporting a "success" on corrupted 0-byte files).   |
|  - UI unlocks, returning control to the user.                                |
+------------------------------------------------------------------------------+


--------------------------------------------------------------------------------
                         DETAILED WORKFLOW EXPLANATION
--------------------------------------------------------------------------------

1. SMART INITIALIZATION
The software is designed to prevent user error before processing begins. When a 
file is selected, the app dynamically detects if it is an image or video. Instead 
of relying on standard Windows libraries (which can lock files and cause crashes), 
it uses FFmpeg to quietly probe the file's metadata, extract the native resolution, 
and auto-fill the Exact Resolution text boxes. 

2. ASYNCHRONOUS ENGINE & TELEMETRY
Upscaling is highly hardware-intensive. To prevent the application from freezing 
("hanging"), the core processing engine is separated from the UI using modern 
Async/Await tasks. While the engine runs invisibly in the background, a dedicated 
telemetry thread directly queries the Windows Performance Counters and NVIDIA-SMI 
to plot real-time hardware utilization (CPU, RAM, GPU) to the dashboard graph.

3. THE "EXACT RESOLUTION" SECRET
AI vision models like Real-ESRGAN are mathematically locked to exact integer 
scaling (2x, 3x, 4x) and cannot natively generate custom dimensions (e.g., 1920x1080). 
To bypass this limitation, the application forces the AI engine to generate an 
oversized, high-fidelity 4x upscale into a temporary file. It then instantly pipes 
that massive file through FFmpeg to cleanly resize it to the user's exact target.

4. THE VIDEO PIPELINE (FRAME-BY-FRAME)
Video upscaling requires a robust 3-step pipeline to avoid VRAM bottlenecking:
  A. Extraction: FFmpeg isolates the audio and rips every individual video frame 
     into a temporary folder as lossless PNGs.
  B. AI Batching: Real-ESRGAN processes the folder. An independent background loop 
     monitors the folder's file count to calculate an accurate ETA for the user.
  C. Hardware Reassembly: Once upscaled, FFmpeg is summoned again. Instead of 
     using the CPU, it routes the workload directly to the GPU's hardware encoder 
     (HEVC_NVENC) to rapidly rebuild the video stream, apply any custom scaling 
     filters, and map the original audio back in.

5. THE TRUTH CHECK
Command-line tools will occasionally fail due to VRAM limitations or codec limits 
(such as exceeding H.264's maximum 4K boundaries) but still generate an empty, 
corrupted 0-byte output file. The software includes a final "Truth Check" that 
audits the output file size to ensure it is valid data before declaring success. 
Corrupted files are safely purged.


Please note that this application is uncensored, works completely offline and does not interact with the internet at all. Therefore, use it responsibly. 
