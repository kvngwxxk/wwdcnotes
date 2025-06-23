# WWDC 2025 Notes

## Table of Contents


1. [Overview](#1-overview)  
2. [Major Updates by Platform](#2-major-updates-by-platform)  
   1. [iOS 26](#21-ios-26)  
   2. [iPadOS 26](#22-ipados-26)  
   3. [macOS 26 (Tahoe)](#23-macos-26-tahoe)  
   4. [watchOS 26](#24-watchos-26)  
   5. [visionOS 26](#25-visionos-26)  
   6. [tvOS 26](#26-tvos-26)  
3. [In-Depth Analysis by Framework & Technology Area](#3-in-depth-analysis-by-framework--technology-area)  
   1. [SwiftUI & UIKit](#31-swiftui--uikit)  
   2. [Swift 6.2](#32-swift-62)  
   3. [Core ML & Apple Intelligence](#33-core-ml--apple-intelligence)  
   4. [Metal 4 & Graphics Technology](#34-metal-4--graphics-technology)  
   5. [ARKit / RealityKit / SceneKit](#35-arkit--realitykit--scenekit)  
   6. [Networking & Communication](#36-networking--communication)  
4. [Appendix](#4-appendix)  
   
   
## 1. Overview

At WWDC 2025, Apple announced unified updates across all major platforms—iOS, iPadOS, macOS, watchOS, visionOS, and tvOS—moving to version 26. Alongside these updates came Xcode 26, Swift 6.2 enhancements, new framework introductions, and a significant revamp of the design system.

This document is a technical note for developers, systematically summarizing major changes per platform and framework, along with key technology concepts, code samples, and migration strategies.

## 2. Platform-Specific Updates

### 2.1 iOS 26

#### 2.1.1 Liquid Glass Design
A consistent translucent glass texture layer creates a sense of depth throughout the system.
- **Applied Areas**: Lock screen, Home screen widgets, Control Center, Notification Center, App Switcher
- **Effect Composition**: Visual depth created using blur levels (ultraThin, regular, thick)
- **Icon Design**: Multi-layer icon generation with Xcode 26 Icon Composer
- **SF Symbols Usage**: Symbol icon examples using `glassEffect` option

#### 2.1.2 Foundation Models Framework
A lightweight AI framework enabling large language model execution on-device.
- **Model Structure**: Up to 3 billion parameters, quantized to 2-bit for on-device operation
- **Key Features**:
  - Text summarization and classification
  - Keyword extraction and sentiment analysis
  - Live Translation, Writing Tools, Genmoji, Image Playground, etc.

- **Usage Example Code**:

- **Basic Text Summarization**:
```swift
import FoundationModels
let model = try FoundationModel.default()
let article = "Apple today announced a major update to its operating systems..."
let prompt = FMPrompt("Summarize the following article:\n\(article)")
let summary: String = try await model.generate(prompt)
print(summary)
```

- **Sentiment Analysis**:
```swift
let sentimentPrompt = FMPrompt("Classify the sentiment of this sentence: I love using this app!")
let sentiment: String = try await model.generate(sentimentPrompt)
print(sentiment) // Expected: Positive
```

- **Keyword Extraction**:
```swift
let keywordPrompt = FMPrompt("Extract keywords from this sentence: Apple Intelligence brings powerful AI features to iOS.")
let keywords: String = try await model.generate(keywordPrompt)
print(keywords) // Expected: Apple Intelligence, AI, iOS
```

- **Handling Long Inputs and Errors**:
```swift
let longText = String(repeating: "This is a long sentence. ", count: 100)
let prompt = FMPrompt("Summarize:\n\(longText)")
do {
    let result = try await model.generate(prompt)
    print(result)
} catch {
    print("Generation failed: \(error.localizedDescription)")
}
```

#### 2.1.3 App Intents & Visual Intelligence Expansion
- **Home Screen Widgets & Control Center**:
  - Action buttons and form input support
  - Improved inline menus and interactions
- **Visual Intelligence**:
  - Automatic shortcut generation based on images and context

#### 2.1.4 CallTranslation API
- **Real-time subtitles for FaceTime**:
  - Voice recognition and auto-translation
- **Third-party App Support**:
  - External VoIP apps can utilize CallKit translation APIs

#### 2.1.5 Apple Games & Game Center Integration
- **Game Center**:
  - Challenges and leaderboard integration
- **Personalized Recommendations**:
  - Auto-suggestions based on play patterns

#### 2.1.6 Declared Age Range API
- **Description**: Controls content access and custom UI based on system-declared age ranges
- **Age Categories**:
  - Returns C0(0–12), C1(13–17), C2(18+)
- **Content Restriction**:
  - Check `UIApplication.shared.requestedAgeRange` to limit UI/content
- **PrivacyInfo Settings Required**:
  - Add `DeclaredAgeRange` key to `NSPrivacyAccessedAPITypes` in `PrivacyInfo.xcprivacy`
  - Example:
    ```json
    {
      "NSPrivacyAccessedAPITypes": [
        {
          "NSPrivacyAccessedAPIType": "DeclaredAgeRange",
          "NSPrivacyAccessedAPITypeReasons": ["Age-based customization"]
        }
      ]
    }
    ```

- **Example Code**:
```swift
import UIKit
if let range = UIApplication.shared.requestedAgeRange {
    switch range {
    case .child:
        // Configure child mode
    case .teen:
        // Configure teen mode
    case .adult:
        // Configure adult mode
    @unknown default:
        break
    }
}
```

### 2.2 iPadOS 26

#### 2.2.1 Liquid Glass Design
- **Applied Areas**: Sidebar, Toolbar, Dock
- **Visual Consistency**: Maintains a unified look throughout the system with translucent glass effects

#### 2.2.2 Menu Bar Introduction & Custom Menus
- **macOS-style Menu Bar**:
  - Supports custom app-specific menus
- **UIKit & SwiftUI API Usage Example**:
```swift
@main
struct MyApp: App {
    var body: some Scene {
        WindowGroup { ContentView() }
            .commands {
                CommandMenu("Custom") {
                    Button("Refresh Data") { viewModel.reload() }
                        .keyboardShortcut("R", modifiers: [.command])
                    Divider()
                    Button("Settings") { showSettings.toggle() }
                        .keyboardShortcut(",", modifiers: [.command])
                }
            }
    }
}
```

#### 2.2.3 Window Management & Multi-Window Enhancements
- **Freeform Window Placement**:
  - Users can adjust window size and position via touch or pointer
- **Stage Manager Improvements**:
  - Offers window grouping and pinning capabilities
- **SwiftUI Scene Activation Example**:
```swift
func openNewScene() {
    let options = UISceneActivationRequestOptions()
    options.requestingScene = UIApplication.shared.connectedScenes.first
    UIApplication.shared.requestSceneSessionActivation(nil, userActivity: nil, options: options) { error in
        print("Scene activation failed: \(error)")
    }
}
```

#### 2.2.4 BGContinuedTask API
- **Guaranteed Long-running Background Tasks**:
  - Ensures execution even after app termination or device lock
- **API Usage Example**:
```swift
import BackgroundTasks
func scheduleCleanupTask() {
    let request = BGProcessingTaskRequest(identifier: "com.example.cleanup")
    request.requiresNetworkConnectivity = false
    request.requiresExternalPower = true
    request.earliestBeginDate = Date(timeIntervalSinceNow: 30 * 60)
    try BGTaskScheduler.shared.submit(request)
}
```

#### 2.2.5 Slide Over & Split View Enhancements
- **Slide Over**:
  - Added gesture support for switching between windows
- **Split View**:
  - Expanded APIs for controlling column width

#### 2.2.6 Apple Pencil & Scribble Updates
- **Reduced Latency**:
  - Pen input latency improved by 20% compared to previous version
- **Custom Scribble Field Usage Example**:
```swift
let scribble = UIScribbleInteraction(delegate: self)
textField.addInteraction(scribble)
```

#### 2.2.7 Quick Note & Notes Integration
- **Quick Note Accessibility**:
  - Instantly activate via swipe from screen corner
- **Automatic Content Recognition**:
  - Automatically supports web links and document previews

### 2.3 macOS 26 (Tahoe)

- **Liquid Glass Design Integration**  
  Translucent glass-textured layers are applied to sidebars, toolbars, menu bars, notification center, and the Dock, providing a sense of depth and a cohesive visual theme.

- **Enhanced Spotlight**  
  - App Intents Execution: Trigger app shortcuts directly from Spotlight  
  - Menu Command Exposure: Use app commands right after `Command + Space`  
  - Swift API Example:  
    ```swift
    import Intents
    let interaction = INInteraction(intent: MyCustomIntent(), response: nil)
    interaction.donate { error in
        if let error = error { print("Donation failed: \(error)") }
    }
    ```

- **Metal 4 Integration**  
  (See details in [3.4](#34-metal-4--graphics-technology))  
  - Sparse resource management via Placement Heap  
  - Reduced build overhead through Dedicated Compiler Queue and Pipeline Object Harvesting  

- **Video Effects API**  
  - Supports real-time motion blur, frame rate conversion, and super resolution upscaling  
  - Compatible with AVFoundation and Core Image filters  
    ```swift
    import VideoEffects
    let request = VideoEffectRequest(type: .superResolution)
    request.inputFrame = sampleBuffer
    let output = try effectProcessor.process(request)
    ```

- **Live Activities Integration in Menu Bar**  
  - Expands iOS Live Activities concept to menu bar widgets  
  - Displays real-time updates using `NSStatusBar` API  
    ```swift
    let item = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
    item.button?.title = "Downloading..."
    ```

- **Enterprise API Enhancements**  
  - Expanded support for Enterprise Single Sign-On (SSO) framework  
  - Added control features for MDM (Mobile Device Management) policies  

- **SceneKit & RealityKit Unified Workflow**  
  - Export RealityKit scenes for Vision Pro  
  - `SceneExporter` API for converting SceneKit models to RealityKit  

### 2.4 watchOS 26

- **Liquid Glass Design Adoption**  
  Translucent glass layers are applied to watch components such as complications, notifications, and the Dock.

- **Relevance API**  
  Expands Smart Stack to Apple Watch, offering complication recommendations based on time, location, and heart rate.

- **Control Widget API**  
  Allows complications to be executed directly from Control Center.  
  Provides custom timelines using `CLKComplicationDataSource`.  
  ```swift
  class MyComplicationProvider: NSObject, CLKComplicationDataSource {
      func getTimelineEntries(for complication: CLKComplication, after date: Date, limit: Int, with handler: @escaping ([CLKComplicationTimelineEntry]?) -> Void) {
          // Logic for generating timeline entries
      }
  }
  ```

- **MapKit Enhancements**  
  Supports navigation, overlays, and POI (Points of Interest) search on the watch using the new `WKInterfaceMap` API.  
  ```swift
  map.addOverlay(MKCircle(center: coordinate, radius: 50))
  ```

- **Workout & HealthKit Improvements**  
  - Added new workout types (e.g., Taekwondo, Surfing), API for defining custom workouts  
  - Provides heart rate distribution graphs and HRV (Heart Rate Variability) analysis support

- **Integration with Vision Pro**  
  Enables Hands-Off control via Vision Pro, showing Watch complications in AR HUD view

- **Siri Watch Face & Auto Switching**  
  - Allows setting automatic transitions between watch faces  
  - Recommends faces based on Siri suggestions

### 2.5 visionOS 26

- **3D UI & Spatial Layout**  
  - `SpatialContainer` API for placing SwiftUI views in 3D space  
  - Supports precise alignment between UI elements using Depth Alignment

- **SwiftUI ↔ RealityKit Integration**  
  - Attach SwiftUI views to RealityKit entities using `ViewAttachmentComponent`  
  - Coordinate conversion utility: `convert(_ point: CGPoint, to entity: Entity)`

- **Model3D & LOD Support**  
  - Use `.model3D(named:)` modifier to manage multiple Levels of Detail (LOD)  
    ```swift
    import RealityKit
    let entity = try Entity.loadModel(named: "Chair", in: .arResources)
    entity.generateCollisionShapes(recursive: true)
    scene.addAnchor(entity)
    ```

- **Environment Occlusion & MeshInstancesComponent**  
  - `OcclusionMaterial` applies occlusion effects from real-world objects  
  - `MeshInstancesComponent` for large-scale instance rendering

- **Multi-user Shared Experience**  
  - Supports up to 8 users simultaneously with `SharedExperienceSession` API  
  - Maintains synchronized anchors and interaction states

- **Persona & Avatar Improvements**  
  - Applies updated Persona models with motion capture  
  - Enhanced facial expressions and eye-blinking animations

- **Persistence API**  
  - Stores spatial position and rotation via `PersistenceManager.save(anchor:)`  
  - Accurately restores position on re-launch

- **Widgets & Quick Actions**  
  - Spatial widgets attachable to surfaces like walls and tables  
  - `QuickActionButton` enables gesture-based interactions

- **Improved Game Input**  
  - 90Hz hand tracking API  
  - Dynamic rendering quality control via `CompositorServices`

- **Mac Streaming & Progressive Immersion**  
  - Stream macOS apps to visionOS with auto layout of UI elements  
  - Adjust detail level based on distance with Progressive Immersion

---

### 2.6 tvOS 26

- **Automatic Sign-In API**  
  - Integrates with iCloud Keychain to auto-fill user account info and streamline login flow

- **Liquid Glass Design Adoption**  
  - Applies translucent glass layers to focus engine UI (lists, cards, notifications), adding depth

- **Improved Default User Profile API**  
  - Automatically saves and restores app state during profile switching

- **Metal 4 & Game Controller Support**  
  - Metal 4 upgrade includes Sparse Resources and GPU-accelerated ray tracing  
  - Game Controller framework now supports advanced haptic events and improved compatibility with MFi and third-party controllers

- **tvOS Widgets**  
  - Allows small widgets on the home screen, defined using SwiftUI's `WidgetConfiguration`

- **Video Player & Multi-room Audio**  
  - `AVPlayer` extension for synchronized multi-room streaming  
  - Improved Picture-in-Picture (PiP) playback experience

## 3. In-Depth Analysis by Framework & Technology Area

### 3.1 SwiftUI & UIKit

#### 3.1.1 Design System: Liquid Glass & Custom Themes

- **Enhanced Material Effects**  
  - Usage guidelines for `Material.ultraThin`, `.thin`, `.regular`, `.thick`  
  - Tips for managing Z-Index when stacking multiple layers  
  - Performance: Monitor GPU rendering cost; use `.drawingGroup()` when needed

- **Custom MaterialEffectView (UIKit)**  
  ```swift
  class GlassView: UIVisualEffectView {
      init() {
          super.init(effect: UIBlurEffect(style: .systemUltraThinMaterial))
          layer.cornerRadius = 16
          clipsToBounds = true
      }
      required init?(coder: NSCoder) { fatalError() }
  }
  ```

- **Theme Colors & Dark Mode**  
  - Use compressed format (extended sRGB) in Asset Catalog to reduce memory  
  - SwiftUI: Dynamic color selection using `@Environment(\.colorScheme)`  
  ```swift
  @Environment(\.colorScheme) var scheme
  var bg: Color { scheme == .dark ? Color("DarkBG") : Color("LightBG") }
  ```

#### 3.1.2 State Management & Data Binding

- **Advanced Use of the Observation Framework**  
  - Tracking changes to `let` vs `var` inside `@Observable` classes  
  - Example of applying service locator pattern via `@Dependency`  
  ```swift
  class AppServices: Observable {
      @Dependency(\.apiClient) var apiClient
  }
  ```

- **Hybrid Patterns with Combine**  
  - Bridge third-party libraries using `.publisher`; compare `async let` vs `.sink()` performance  
  - Prevent memory leaks: manage `AnyCancellable` via `store(in:)`

- **Computed Property Binding**  
  - Two-way binding of derived data with `Binding(get:set:)`  
  ```swift
  struct ToggleView: View {
      @State private var isOn = false
      var binding: Binding<String> {
          Binding(
              get: { isOn ? "On" : "Off" },
              set: { isOn = $0 == "On" }
          )
      }
      var body: some View {
          TextField("State", text: binding)
      }
  }
  ```

#### 3.1.3 Advanced Layout & Animation

- **Custom Layout Implementation**  
  - Use of `cache` parameter to improve layout performance  
  - Tips for handling dynamic view size changes

- **Grid API & Adaptive Layout**  
  - Responsive layout with `LazyVGrid(columns: [GridItem(.adaptive(minimum: 100))])`

- **Physics-based Animation**  
  - Guide for tuning `spring(response:dampingFraction:blendDuration:)`  
  - Example: `animation(.interactiveSpring(), value: state)`

- **View Transitions**  
  - Combined effects: `.transition(.move(edge: .bottom).combined(with: .opacity))`

#### 3.1.4 UIKit Integration & Compatibility

- **UIHostingConfiguration**  
  - Embed SwiftUI in views like `UICollectionViewListCell`  
  ```swift
  cell.contentConfiguration = UIHostingConfiguration { ContentView() }
  ```

- **ScenePhase Synchronization**  
  - Strategy for syncing SwiftUI `@Environment(\.scenePhase)` with `UIApplicationDelegate`

- **Adaptive UI Composition**  
  - Layout branching based on size class: `@Environment(\.horizontalSizeClass)`

#### 3.1.5 Accessibility & Internationalization

- **Advanced VoiceOver Support**  
  - Map custom gestures using `accessibilityAction(named:_:)`  
  - Manage focus using `AccessibilityFocusState`

- **Dynamic Type & Localization**  
  - Support large text with `@ScaledFont`  
  - Preserve string formatting during translation:  
    `.formatted(.number.precision(.fractionLength(2)))`

#### 3.1.6 Performance Optimization Tips

- **View Identity Management**  
  - Avoid excessive use of `id(UUID())`; prefer stable unique identifiers

- **Minimize Redraws**  
  - Use `EquatableView` wrapper to skip rendering on state invariance

- **Using Instruments**  
  - Analyze view lifecycle with SwiftUI Rendering Timeline and Memory Graph

### 3.2 Swift 6.2

#### 3.2.1 Advanced Concurrency Extensions

- **`@concurrent` Functions**  
  - Explicit marker for concurrent execution; provides control over reentrancy  
  ```swift
  struct MatrixMultiplication {
      @concurrent func multiply(_ a: [[Int]], _ b: [[Int]]) async -> [[Int]] {
          // Perform concurrent partitioned tasks
      }
  }
  ```

- **Actor Reentrancy Policy**  
  - Use `@actorIndependent` for safe external access  
  - Ensures safety for `nonisolated` methods

- **Structured Concurrency Improvements**  
  - `async let` now supports `throwing` parallel bindings  
  - `Task.group` optimizations for better cancellation and failure handling

#### 3.2.2 Memory & Type Safety

- **Inline Arrays**  
  - Fixed-size arrays allocated on the stack to avoid heap usage  
  - Example: `inline Array<UInt8, 16>`

- **`Span<T>`**  
  - Provides views into contiguous memory buffers with `withUnsafeSpan` utility  
  ```swift
  func processBuffer<T>(data: Span<T>) {
      for element in data {
          // Processing logic
      }
  }
  ```

- **Move-only Type Support**  
  - Declare copy-restricted structures using `@moveOnly` for enhanced safety

#### 3.2.3 Interoperability & Build Tools

- **C++/Java FFI Enhancements**  
  - Automatic bridging with `import CxxModule` and `import JavaModule`  
  ```swift
  import CxxWX
  let widget = WXWidget(options: ...)
  ```

- **Swift Containerization**  
  - Enables compiling Swift inside Docker containers  
  - Adds `swift package export-docker` command

- **`swiftly` CLI & Build Performance**  
  - Parallel test execution: `swiftly test --parallel`  
  - Easy custom compiler flags like `-Xcompiler -fast-math`

#### 3.2.4 Package Manager & ABI Stability

- **SwiftPM Dynamic Library Support**  
  - Declare modules for runtime loading using `library(type: .dynamic)`

- **Versioned ABI**  
  - Strengthens ABI layering across modules  
  - Adds rolling update test capabilities

#### 3.2.5 Compiler Directives & Concurrency Verification

- **New Compiler Directives**

  - `#expressionAs`: Force expression to be cast to a specific type at compile-time  
    ```swift
    let x = #expressionAs(Int.self, "123")
    ```

  - `#compileTime`: Conditional compilation or constant evaluation  
    ```swift
    let y = #compileTime(1 + 2) // 3
    ```

  - `@backDeployed`: Mark functions to run on older OS versions  
    ```swift
    @available(iOS 17, *)
    @backDeployed(before: iOS 16)
    public func doLegacyThing() { ... }
    ```

- **Strict Concurrency Checking Levels**  
  - `minimal`: Detects major actor isolation violations  
  - `targeted`: Focuses on Apple API-related warnings  
  - `complete`: Performs strict checking on all async call flows

- **Example Application Strategy**
  ```swift
  // Example: Apply in Xcode project build settings
  // SWIFT_CONCURRENCY_CHECKS = complete
  ```

  - Start with `targeted` or `minimal`, then gradually migrate to `complete`

### 3.3 Core ML & Apple Intelligence

> ℹ️ **Note**: Some features described in this section—such as Apple Intelligence, Writing Tools, etc.—are currently only available within Apple’s native apps and system services as of WWDC 2025. **Third-party developers do not have direct API access**. Public APIs are limited to FoundationModels, BNNS, Vision, CreateML, and a few others; Apple Intelligence features are not accessible in general-purpose apps.

#### 3.3.1 Foundation Models (AJL)

- **Tool Calling API**  
  Enables calling internal functions within LLMs and defining custom workflows.  
  ```swift
  import FoundationModels
  let model = try FoundationModel.default()
  let prompt = FMPrompt("Translate to Korean: Hello, world!")
  let result: String = try await model.generate(prompt)
  print(result)
  ```

- **Guided Generation**  
  - Supports multi-turn conversations and structured function-style responses  
  - Includes examples of boxed response formats like JSON, XML

#### 3.3.2 BNNS Graph API

- **Custom Neural Network Graphs**  
  Create `BNNSGraph` objects and define layer ordering.  
  ```swift
  import BNNS
  let graph = BNNSGraph()
  let conv = BNNSFilterCreateConvolutionLayer(graph, /* configuration */)
  BNNSGraphCompile(graph)
  BNNSGraphEvaluate(graph)
  ```

- **Performance Tuning**  
  Optimized for hardware via CPU SIMD and vectorization options

#### 3.3.3 Vision Framework

- **Document & Text Recognition**  
  Use `VNDocumentCameraViewController` to scan documents and integrate OCR pipeline.  
  ```swift
  let scanner = VNDocumentCameraViewController()
  scanner.delegate = self
  ```

- **Smudge Detection**  
  Identify smudges by combining `VNDetectHumanHandPoseRequest` and `VNDetectContoursRequest`

#### 3.3.4 SpeechAnalyzer

- **Real-time Voice Analysis**  
  Monitor signal strength, pitch, and text transformation using `SpeechAnalyzer(sessionConfiguration:)`  
  ```swift
  import Speech
  let analyzer = try SpeechAnalyzer()
  analyzer.delegate = self
  analyzer.startAnalysis()
  ```

- **Noise Profiling**  
  Supports noise reduction and Voice Activity Detection (VAD)

#### 3.3.5 Metal ML & MPSGraph

- **Tensor Operations & Model Execution**  
  Compose mathematical operations per layer using `MPSGraph`  
  ```swift
  import MetalPerformanceShadersGraph
  let graph = MPSGraph()
  let x = graph.placeholder(shape: [1, 224, 224, 3], dataType: .float32)
  let y = graph.convolution2D(x, weights: weightTensor, descriptor: descriptor)
  ```

- **On-device Fine-tuning with GPU Acceleration**  
  Use `GraphTrainingContext` to support efficient model updates directly on the device

#### 3.3.6 MLX & Create ML Adapter

- **LLM Fine-tuning Library (MLX)**  
  Includes prompt templates, tokenizer integration, and hyperparameter tuning tools

- **Create ML Adapter**  
  Automates model training and export using Swift API  
  ```swift
  import CreateML
  let classifier = try MLTextClassifier(trainingData: data)
  try classifier.write(to: URL(fileURLWithPath: "MyClassifier.mlmodel"))
  ```

### 3.4 Metal 4 & Graphics Technology

#### 3.4.1 Placement Heap & Sparse Resources

- **Placement Heap**  
  - Dynamically allocate and deallocate textures and buffers from a memory pool  
  - Reduces CPU/GPU synchronization overhead

- **Sparse Resource Binding**  
  - Load large assets in small tile-based chunks to optimize memory usage  
  ```swift
  let heapDescriptor = MTLHeapDescriptor()
  heapDescriptor.size = 64 * 1024 * 1024
  let heap = device.makeHeap(descriptor: heapDescriptor)!

  let textureDesc = MTLTextureDescriptor.texture2DDescriptor(pixelFormat: .rgba8Unorm,
                                                             width: 4096,
                                                             height: 4096,
                                                             mipmapped: false)
  textureDesc.storageMode = .private
  textureDesc.heap = heap

  let texture = heap.makeTexture(descriptor: textureDesc)!
  ```

#### 3.4.2 Dedicated Compiler Queue & Pipeline Object Harvesting

- **Dedicated Compiler Queue**  
  - Offloads runtime shader compilation to a separate queue to stabilize frame rates

- **Pipeline Object Harvesting**  
  - Automatically collects pipeline state objects used during app execution to create optimized bundles  
  ```swift
  let library = try device.makeLibrary(source: shaderSource, options: nil)

  let pipelineDesc = MTLRenderPipelineDescriptor()
  pipelineDesc.vertexFunction = library.makeFunction(name: "vertex_main")
  pipelineDesc.fragmentFunction = library.makeFunction(name: "fragment_main")

  let pipelineState = try device.makeRenderPipelineState(descriptor: pipelineDesc)
  ```

#### 3.4.3 MetalFX & GPU-Accelerated FX

- **Upscale & Denoise**  
  Use `MTLFXUpscaler` and `MTLFXDenoiser` objects for high-resolution and noise reduction processing  
  ```swift
  let upscaler = device.makeFXUpscaler(descriptor: upscalerDescriptor,
                                       sourceTexture: input,
                                       destinationTexture: output)
  upscaler?.encode(commandBuffer: cb)
  ```

- **Efficient Memory Management**  
  - Introduces intermediate buffer reuse and command buffer pooling patterns

#### 3.4.4 Ray Tracing & Acceleration Structures

- **Ray Tracing Intersection Shaders**  
  Define `intersection_function` and configure `MTLAccelerationStructureDescriptor`

- **Acceleration Structure Build Optimization**  
  Choose build mode using `MTLAccelerationStructureControlOptions` (e.g., fast build vs fast trace)

#### 3.4.5 Debugging & Profiling Tools

- **Metal System Trace**  
  - Instruments support for GPU capture and timeline analysis

- **Shader Profiler**  
  - View runtime shader performance, cycle counts, and register usage

### 3.5 ARKit

#### 3.5.1 RoomPlan & Anchors

- **Room Anchor (RoomPlan)**  
  Automatically detects walls, floors, and ceilings of indoor spaces and generates anchors.  
  ```swift
  import RoomPlan
  let config = RoomPlan.Configuration()
  session.run(config)
  ```

- **Handling Custom Anchors**  
  Manage app-specific anchor metadata using `ARExperienceAnchor`

#### 3.5.2 Plane Detection & Tracking

- **Slanted Plane Detection**  
  Detects inclined surfaces such as table edges or slanted walls

- **TrackingState API**  
  Monitor real-time tracking state using:  
  `scene.session.currentFrame?.camera.trackingState`

#### 3.5.3 Object & Face Tracking

- **Reference Object Tracking**  
  Load and continuously track objects using `ARReferenceObject`  
  ```swift
  let object = try Entity.load(named: "MyARObject")
  let anchor = try ARObjectAnchor(referenceObject: referenceObject, transform: matrix)
  ```

- **Face Tracking & BlendShapes**  
  Observe facial expressions and blend shape values with `ARFaceAnchor`, and visualize with `FaceMesh`

### 3.6 RealityKit

#### 3.6.1 SwiftUI Integration & Entity Lifecycle

- **ViewAttachmentComponent**  
  Attach SwiftUI views to RealityKit entities  
  ```swift
  let viewEntity = ModelEntity()
  viewEntity.components[ViewAttachmentComponent.self] = .init(rootView: ContentView())
  ```

- **Entity Lifecycle Management**  
  Handle animation and state synchronization per frame using `SceneEvents.Update`

#### 3.6.2 Mesh & Rendering Features

- **MeshInstancesComponent**  
  Optimized rendering for large numbers of instances

- **OcclusionMaterial**  
  Enhances realism by rendering occlusion effects from real-world environments

### 3.7 SceneKit

#### 3.7.1 SceneExporter & RealityKit Migration

- **SceneExporter Utility**  
  Export `SCNScene` into RealityKit `.reality` file format  
  ```swift
  import SceneKit
  let scene = try SCNScene(named: "scene.scn")
  SceneExporter.export(scene, to: URL(fileURLWithPath: "scene.reality"))
  ```

#### 3.7.2 Material Mapping

- Describes automatic conversion rules between SceneKit’s `SCNMaterial` and RealityKit’s `Material` properties

### 3.8 Networking & Communication

This section summarizes the latest improvements to the networking stack and communication frameworks.

#### 3.8.1 HTTP/3 & Network Performance

- **HTTP/3 Native Support**  
  - Based on the QUIC protocol, supports multiplexing and header compression  
  - Enable via `allowsHTTP3` in `URLSessionConfiguration`  
  ```swift
  let config = URLSessionConfiguration.default
  config.allowsHTTP3 = true
  let session = URLSession(configuration: config)
  ```

- **Native Instruments Tracing**  
  - Tools to analyze HTTP/3 connections, stream states, and packet transmission

#### 3.8.2 Network.framework & Wi‑Fi Aware

- **Network.framework async/await**  
  - Asynchronous APIs for `NWConnection` and `NWListener`  
  ```swift
  let conn = NWConnection(host: "example.com", port: 443, using: .tls)
  await conn.start()
  let data = try await conn.receive(minimumIncompleteLength: 1, maximumLength: 65536)
  ```

- **Wi‑Fi Aware (NAN)**  
  - Supports direct communication between nearby devices using the `NANSession` API

#### 3.8.3 Background Assets & URLSession

- **Download/Upload via BackgroundAssets**  
  - Reliable transfer of large files using `URLSession`'s background sessions  
  ```swift
  let backgroundConfig = URLSessionConfiguration.background(withIdentifier: "com.example.bg")
  let backgroundSession = URLSession(configuration: backgroundConfig, delegate: self, delegateQueue: nil)
  ```

- **Resume & Cancel Support**  
  - Usage of `downloadTask(withResumeData:)` and `cancel(byProducingResumeData:)` patterns

#### 3.8.4 Real-time Communication & WebSocket

- **URLSessionWebSocketTask**  
  - Examples of `send(_:completionHandler:)` and `receive(completionHandler:)`  
  ```swift
  let webSocket = session.webSocketTask(with: URL(string: "wss://echo.websocket.org")!)
  webSocket.resume()
  webSocket.send(.string("Hello")) { error in /* ... */ }
  ```

- **Server Push Notifications**  
  - HTTP/2 `push` events using `URLSession.delegate`'s `didReceive challenge`

#### 3.8.5 Security & Authentication

- **TLS 1.3 Optimization**  
  - Configure custom encryption suites with `NWProtocolTLS.Options`

- **OAuth2 & Authentication Flows**  
  - OAuth2 login example using `ASWebAuthenticationSession`  
  ```swift
  let authSession = ASWebAuthenticationSession(url: authURL, callbackURLScheme: callbackScheme) { callbackURL, error in
      // Handle authentication result
  }
  authSession.start()
  ```

## 4. Appendix & References

### WWDC 2025 Key Session List & IDs

| Platform / Topic            | Session Title & Number                            |
|----------------------------|----------------------------------------------------|
| Foundation Models           | Meet the Foundation Models (WWDC25-286)           |
| App Intents advances        | Explore new advances in App Intents (WWDC25-275)  |
| SwiftUI Updates             | What’s new in SwiftUI (WWDC25-256)                |
| Swift Concurrency           | Embracing Swift concurrency (WWDC25-250)          |
| Metal 4 Introduction        | What’s new in Metal (WWDC25-262)                  |
| Rich Text in SwiftUI        | Cook up a rich text experience (WWDC25-243)       |

> 📝 **References**: This document is based on the official content presented at [Apple WWDC 2025](https://developer.apple.com/wwdc25/), along with supplemental developer session videos and materials provided by Apple.
