Empower your visual research with neuralresolv. Harnessing local LLMs, predictive modelling, and neural networks, it delivers exceptional image and video upscaling. Enhance low-resolution media and uncover vital details in your data with precision, all processed securely offline.

### 🧠 How It Works: Under the Hood

```mermaid
flowchart TD
    %% Styling
    classDef default fill:#2b2b2b,stroke:#a8a8a8,stroke-width:2px,color:#fff;
    classDef input fill:#1e4c70,stroke:#2b7bb8,stroke-width:2px,color:#fff;
    classDef aiEngine fill:#5b2a86,stroke:#8a40c9,stroke-width:2px,color:#fff;
    classDef ffmpeg fill:#347836,stroke:#4caf50,stroke-width:2px,color:#fff;
    classDef io fill:#8f4b0d,stroke:#cf7017,stroke-width:2px,color:#fff;

    %% --- GUI Layer ---
    subgraph UI ["🖥️ GUI & Initialization (VB.NET Thread)"]
        A([User Selects Media]):::input --> B[Auto-Detect Extension & Read Metadata]
        B --> C{Scaling Mode Selected?}
        C -->|Ratio| D[Lock Variables]
        C -->|Exact Resolution| E[Auto-calculate Aspect Ratio]
        D --> F([Trigger Async Task]):::input
        E --> F
    end

    %% --- Telemetry Layer ---
    subgraph Telemetry ["📊 Live Hardware Telemetry"]
        T1[PerformanceCounters] -->|CPU & RAM %| T3[Real-time Chart Generator]
        T2[NVIDIA-SMI] -->|GPU Utilization %| T3
    end

    %% --- Core Processing Pipeline ---
    subgraph CoreEngine ["⚙️ Asynchronous Processing Pipeline"]
        F --> G[Lock UI & Init Progress Trackers]
        G --> H{Is Media Type Video or Image?}

        %% Image Pipeline
        subgraph ImagePipeline ["🖼️ Image Processing"]
            H -->|Image| I1{Scaling Method?}
            I1 -->|Scale by Ratio| I2[Real-ESRGAN: Upscale Native Ratio]:::aiEngine
            
            I1 -->|Exact Resolution| I3[Real-ESRGAN: Force 4x Upscale to Temp]:::aiEngine
            I3 --> I4[FFmpeg: Resize Temp to Exact Target Dimensions]:::ffmpeg
        end

        %% Video Pipeline
        subgraph VideoPipeline ["🎬 Video Processing (Frame-by-Frame)"]
            H -->|Video| V1[FFmpeg: Extract Audio & Target FPS]:::ffmpeg
            V1 --> V2[FFmpeg: Extract All Frames to Temp Directory]:::ffmpeg
            
            V2 --> V3[Real-ESRGAN: Batch Upscale Directory]:::aiEngine
            V3 -.->|Background Task Monitors Folder| P1[Calculate ETA based on Extracted Files]
            
            V3 --> V4{Scaling Method?}
            V4 -->|Scale by Ratio| V5[FFmpeg + NVENC: Reassemble Video & Audio]:::ffmpeg
            V4 -->|Exact Resolution| V6[FFmpeg + NVENC: Resize Frames & Reassemble]:::ffmpeg
        end
    end

    %% --- Output Layer ---
    subgraph Finalization ["💾 Finalization"]
        I2 --> O1[Temporary File Cleanup]
        I4 --> O1
        V5 --> O1
        V6 --> O1
        
        O1 --> O2{Truth Check: Does File Exist & > 0KB?}
        O2 -->|Yes| O3([Success: Save File]):::io
        O2 -->|No| O4([Fail: Delete Corrupt Output & Throw Error]):::io
    end

Please note that this application is uncensored, works completely offline and does not interact with the internet at all.

Therefore, use it responsibly. 
