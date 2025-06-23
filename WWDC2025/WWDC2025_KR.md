# WWDC 2025 Notes
## 목차
1. [개요](#1-개요)
2. [플랫폼별 주요 업데이트](#2-플랫폼별-주요-업데이트)
   1. [iOS 26](#21-ios-26)
   2. [iPadOS 26](#22-ipados-26)
   3. [macOS 26 (Tahoe)](#23-macos-26-tahoe)
   4. [watchOS 26](#24-watchos-26)
   5. [visionOS 26](#25-visionos-26)
   6. [tvOS 26](#26-tvos-26)
3. [프레임워크 및 기술 분야별 심층 분석](#3-프레임워크-및-기술-분야별-심층-분석)
   1. [SwiftUI & UIKit](#31-swiftui-uikit)
   2. [Swift 6.2](#32-swift-6-2)
   3. [Core ML & Apple Intelligence](#33-core-ml-apple-intelligence)
   4. [Metal 4 & 그래픽 기술](#34-metal-4-그래픽-기술)
   5. [ARKit / RealityKit / SceneKit](#35-arkit-realitykit-scenekit)
   6. [네트워킹 & 통신](#36-네트워킹-통신)
4. [부록](#4-부록)


## 1. 개요
 WWDC 2025에서는 iOS, iPadOS, macOS, watchOS, visionOS, tvOS 전반에 걸쳐 버전 26으로 통합된 업데이트가 발표되었으며, 이와 함께 Xcode 26, Swift 6.2 개선, 신규 프레임워크 도입, 디자인 시스템 혁신 등 다양한 변화가 공개되었습니다.

본 문서는 각 플랫폼과 프레임워크의 주요 변경 사항을 중심으로, 핵심 기술 개념, 코드 예시, 마이그레이션 전략까지 체계적으로 정리한 개발자용 기술 노트입니다.


## 2. 플랫폼별 주요 업데이트

### 2.1 iOS 26

#### 2.1.1 Liquid Glass 디자인
시스템 전반에 걸쳐 일관된 반투명 유리 질감의 레이어를 통한 깊이감 연출
- **적용 영역**: 잠금 화면, 홈 화면 위젯, 제어 센터, 알림 센터, 앱 전환기
- **효과 구성**: 블러 레벨(ultraThin, regular, thick)을 활용한 시각적 깊이감 제공
- **아이콘 디자인**: Xcode 26 Icon Composer 활용 다계층 아이콘 생성 방법
- **SF Symbols 활용**: `glassEffect` 옵션을 이용한 심볼 아이콘 예시

#### 2.1.2 Foundation Models 프레임워크
온디바이스에서 대규모 언어 모델 실행을 가능케 하는 경량화된 AI 프레임워크 제공
- **모델 구조**: 최대 30억 매개변수를 2비트 양자화로 경량화하여 온디바이스 구동
- **주요 기능**:
  - 텍스트 요약 및 분류
  - 키워드 추출 및 감정 분석
  - Live Translation, Writing Tools, Genmoji, Image Playground 등
- **활용 예시 코드**:
- **기본 텍스트 요약**:
```swift
import FoundationModels
let model = try FoundationModel.default()
let article = "Apple today announced a major update to its operating systems..."
let prompt = FMPrompt("Summarize the following article:
\(article)")
let summary: String = try await model.generate(prompt)
print(summary)
```

- **감정 분석**:
```swift
let sentimentPrompt = FMPrompt("Classify the sentiment of this sentence: I love using this app!")
let sentiment: String = try await model.generate(sentimentPrompt)
print(sentiment) // Expected: Positive
```

- **키워드 추출**:
```swift
let keywordPrompt = FMPrompt("Extract keywords from this sentence: Apple Intelligence brings powerful AI features to iOS.")
let keywords: String = try await model.generate(keywordPrompt)
print(keywords) // Expected: Apple Intelligence, AI, iOS
```

- **긴 입력 및 에러 처리**:
```swift
let longText = String(repeating: "This is a long sentence. ", count: 100)
let prompt = FMPrompt("Summarize:
\(longText)")
do {
    let result = try await model.generate(prompt)
    print(result)
} catch {
    print("Generation failed: \(error.localizedDescription)")
}
```

#### 2.1.3 App Intents 및 Visual Intelligence 확장
- **홈 화면 위젯 및 제어 센터**:
  - 액션 버튼 제공 및 폼 입력 지원
  - 인라인 메뉴 및 인터랙션 개선
- **Visual Intelligence**:
  - 이미지 및 컨텍스트 기반 자동 단축어 생성

#### 2.1.4 CallTranslation API
- **FaceTime 통화 실시간 자막 번역**:
  - 음성 인식 및 자동 번역 제공
- **서드파티 앱 지원**:
  - CallKit 번역 API를 외부 VoIP 앱에서 활용 가능

#### 2.1.5 Apple Games 및 Game Center 통합
- **Game Center**:
  - 챌린지 기능 및 리더보드 통합
- **개인화 추천**:
  - 플레이 패턴 기반 자동 게임 추천

#### 2.1.6 Declared Age Range API
- **설명**: 시스템 제공 연령대 정보를 기반으로 콘텐츠 접근 제어 및 맞춤형 UI 구성
- **연령대 식별**:
  - C0(0–12), C1(13–17), C2(18+) 반환
- **콘텐츠 제어**:
  - `UIApplication.shared.requestedAgeRange` 검사 후 UI·콘텐츠 제한
- **PrivacyInfo 설정 필요**:
  - `PrivacyInfo.xcprivacy` 파일 내 `NSPrivacyAccessedAPITypes` 항목에 `DeclaredAgeRange` 키 추가 필요
  - 예시:
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
- **코드 예시**:
```swift
import UIKit
if let range = UIApplication.shared.requestedAgeRange {
    switch range {
    case .child:
        // 어린이 모드 구성
    case .teen:
        // 청소년 모드 구성
    case .adult:
        // 성인 모드 구성
    @unknown default:
        break
    }
}
```


### 2.2 iPadOS 26

#### 2.2.1 Liquid Glass 디자인
- **적용 영역**: 사이드바, 툴바, 독(Dock)
- **디자인 일관성**: 시스템 전체에 반투명 유리 효과로 시각적 통일성 유지

#### 2.2.2 메뉴 바 도입 및 커스텀 메뉴
- **macOS 스타일 메뉴 바**:
  - 앱별 커스텀 메뉴 지원
- **UIKit 및 SwiftUI API 활용 예시**:
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

#### 2.2.3 창 관리 및 멀티 윈도우 강화
- **자유형 창 배치**:
  - 사용자가 터치나 포인터로 창 크기 및 위치 조정 가능
- **Stage Manager 개선**:
  - 창 그룹핑 및 고정 기능 제공
- **SwiftUI Scene 활성화 코드 예시**:
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
- **장시간 백그라운드 작업 보장**:
  - 앱 종료 또는 기기 잠금 상태에서도 작업 실행 보장
- **API 활용 코드 예시**:
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

#### 2.2.5 Slide Over & Split View 개선
- **Slide Over**:
  - 창 전환 제스처 추가
- **Split View**:
  - 컬럼 너비 조정 API 확대

#### 2.2.6 Apple Pencil & Scribble 업데이트
- **지연 시간(latency) 감소**:
  - 기존 대비 펜 입력 지연 시간 20% 개선
- **커스텀 Scribble 필드 활용 코드 예시**:
```swift
let scribble = UIScribbleInteraction(delegate: self)
textField.addInteraction(scribble)
```

#### 2.2.7 Quick Note & Notes 통합
- **Quick Note 접근성**:
  - 화면 모서리 스와이프 시 빠르게 활성화 가능
- **자동 콘텐츠 인식**:
  - 웹 링크 및 문서 미리보기 자동 지원

### 2.3 macOS 26 (Tahoe)

- **Liquid Glass 디자인 통합**  
  사이드바, 도구 막대(Toolbar), 메뉴 바, 알림 센터, Dock까지 반투명 유리 질감의 레이어가 적용되어 깊이감과 일관된 비주얼 테마를 제공

- **Spotlight 고도화**  
  - App Intents 실행 기능: Spotlight에서 직접 앱 단축어(Shortcut)를 트리거  
  - 메뉴 명령 노출: `Command + Space` 후 바로 앱 내 커맨드 사용 가능  
  - Swift API 예시:  
  ```swift
  import Intents
  let interaction = INInteraction(intent: MyCustomIntent(), response: nil)
  interaction.donate { error in
      if let error = error { print("Donation failed: \(error)") }
  }
  ```

- **Metal 4 탑재**  
  (자세한 내용은 [3.4](#34-metal-4-그래픽-기술) 참조)  
  - Placement Heap을 통한 Sparse Resource 관리  
  - Dedicated Compiler Queue 및 Pipeline Object Harvesting으로 빌드 오버헤드 감소  

- **Video Effects API**  
  - 실시간 모션 블러, 프레임 레이트 변환, 슈퍼 해상도 업스케일링 지원  
  - AVFoundation 및 Core Image 필터 호환  
  ```swift
  import VideoEffects
  let request = VideoEffectRequest(type: .superResolution)
  request.inputFrame = sampleBuffer
  let output = try effectProcessor.process(request)
  ```

- **Live Activities 메뉴 바 연동**  
  - iOS의 Live Activities 개념을 메뉴 바 위젯으로 확장  
  - `NSStatusBar` API 통해 실시간 정보 업데이트 표시  
  ```swift
  let item = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
  item.button?.title = "Downloading..."
  ```

- **Enterprise API 강화**  
  - Enterprise Single Sign-On(SSO) 프레임워크 지원 확대  
  - MDM(모바일 디바이스 관리) 정책 제어 기능 추가  

- **SceneKit & RealityKit 통합 워크플로우**  
  - Vision Pro용 RealityKit Scene export 옵션  
  - SceneKit 모델을 RealityKit로 변환하는 `SceneExporter` API 제공

### 2.4 watchOS 26

- **Liquid Glass 디자인 반영**  
  워치용 컴포넌트(콤플리케이션, 알림, 독 등)에 반투명 유리 레이어 적용

- **Relevance API**  
  Smart Stack을 워치로 확장, 시간·위치·심박수 기반 콤플리케이션 추천

- **Control Widget API**  
  제어 센터에서 워치 콤플리케이션 바로 실행 지원, `CLKComplicationDataSource`로 커스텀 타임라인 제공
  ```swift
  class MyComplicationProvider: NSObject, CLKComplicationDataSource {
      func getTimelineEntries(for complication: CLKComplication, after date: Date, limit: Int, with handler: @escaping ([CLKComplicationTimelineEntry]?) -> Void) {
          // 타임라인 엔트리 생성 로직
      }
  }
  ```

- **MapKit 확장**  
  워치에서 경로 안내, 오버레이, POI 탐색 지원, `WKInterfaceMap` 새로운 API
  ```swift
  map.addOverlay(MKCircle(center: coordinate, radius: 50))
  ```

- **Workout & HealthKit 기능 강화**  
  - 새로운 운동 타입(태권도·서핑 등) 추가, 커스텀 워크아웃 정의 API  
  - 심박수 분포 그래프 제공, HRV 분석 지원

- **Vision Pro 연동**  
  - Vision Pro Hands-Off 컨트롤(워치 콤플리케이션 뷰를 AR HUD로 표시)

- **Siri Watch Face & 전환**  
  - 시계 페이스간 자동 전환 조건 설정, Siri 추천 기반 페이스 제안



### 2.5 visionOS 26

- **3D UI & Spatial Layout**  
  - SwiftUI 뷰를 3차원 공간에 배치하는 `SpatialContainer` API  
  - 뎁스 얼라인먼트(Depth Alignment)로 UI 요소 간 정확한 정렬 지원

- **SwiftUI ↔ RealityKit 통합**  
  - `ViewAttachmentComponent`로 SwiftUI 뷰를 RealityKit 엔티티에 부착  
  - 좌표 변환 유틸리티 `convert(_ point: CGPoint, to entity: Entity)` 제공

- **Model3D & LOD 지원**  
  - `.model3D(named:)` 모디파이어로 다양한 LOD(Level of Detail) 애셋 처리  
  ```swift
  import RealityKit
  let entity = try Entity.loadModel(named: "Chair", in: .arResources)
  entity.generateCollisionShapes(recursive: true)
  scene.addAnchor(entity)
  ```

- **환경 오클루전 & MeshInstancesComponent**  
  - 실제 물체에 가려짐(Occlusion) 효과를 적용하는 `OcclusionMaterial`  
  - 대량 인스턴스 렌더링용 `MeshInstancesComponent`

- **Multi-user Shared Experience**  
  - `SharedExperienceSession` API로 최대 8명 동시 경험 지원  
  - 동기화된 앵커(anchor) 및 상호작용 상태 유지

- **Persona & Avatar 개선**  
  - 모션 캡처 기반 최신 Persona 모델 적용  
  - 표정·눈 깜빡임 애니메이션 품질 향상

- **Persistence API**  
  - 공간 상 위치·회전 상태 저장 `PersistenceManager.save(anchor:)`  
  - 재실행 시 정확한 위치 복원

- **위젯 & Quick Actions**  
  - 벽·탁자 등 평면에 부착 가능한 공간 위젯  
  - `QuickActionButton`으로 손 제스처 기반 인터랙션 지원

- **게임 입력 개선**  
  - 손 추적 90Hz API  
  - `CompositorServices`로 렌더링 품질 동적 조정  

- **Mac 스트리밍 & Progressive Immersion**  
  - macOS 앱을 visionOS로 스트리밍, UI 요소 자동 레이아웃  
  - Progressive Immersion: 거리 기반 디테일 조정



### 2.6 tvOS 26

- **Automatic Sign-In API**  
  iCloud 키체인 연동으로 사용자 계정 정보를 자동으로 가져와 로그인 흐름 자동화 지원

- **Liquid Glass 디자인 반영**  
  tvOS용 포커스 엔진 UI(리스트·카드·알림)에도 반투명 유리 레이어 적용으로 깊이감 연출

- **Default User Profile API 개선**  
  다중 사용자 프로필 전환 시 애플리케이션 상태 자동 저장 및 복원 지원

- **Metal 4 & Game Controller 지원**  
  - Metal 4 업그레이드: Sparse Resource, GPU 가속 레이트레이싱 탑재  
  - Game Controller 프레임워크 확장: 특수 햅틱 이벤트, MFi 및 서드파티 컨트롤러 호환성 강화

- **tvOS Widgets**  
  홈 화면에 작은 위젯 배치 가능, SwiftUI `WidgetConfiguration`으로 직접 정의

- **Video Player & Multi-room Audio**  
  `AVPlayer` 확장으로 다중 방(Room) 스트리밍 동기화, Picture-in-Picture(PiP) 재생 개선



## 3. 프레임워크 및 기술 분야별 심층 분석

### 3.1 SwiftUI & UIKit

#### 3.1.1 디자인 시스템: Liquid Glass와 커스텀 테마
- **Material 효과 심화**  
  - `Material.ultraThin`, `.thin`, `.regular`, `.thick` 레벨별 사용 가이드라인  
  - 다중 레이어 중첩 시 Z-Index 관리 팁  
  - 성능: GPU 렌더링 비용 모니터링, 필요시 `.drawingGroup()` 사용
- **커스텀 MaterialEffectView** (UIKit)  
  ```swift
  class GlassView: UIVisualEffectView {
      init() {
          super.init(effect: UIBlurEffect(style: .systemUltraThinMaterial))
          layer.cornerRadius = 16; clipsToBounds = true
      }
      required init?(coder: NSCoder) { fatalError() }
  }
  ```
- **테마 색상 & 다크 모드**  
  - Asset Catalog에서 압축 포맷(extended SRGB) 사용으로 메모리 절감  
  - SwiftUI: `@Environment(\.colorScheme)`에 따른 동적 색상 선택 예시
  ```swift
  @Environment(\.colorScheme) var scheme
  var bg: Color { scheme == .dark ? Color("DarkBG") : Color("LightBG") }
  ```

#### 3.1.2 상태 관리 및 데이터 바인딩
- **Observation 프레임워크 고급 활용**  
  - `@Observable` 클래스 내 `let` vs `var` 변경 추적 정책  
  - `@Dependency`로 서비스 로케이터 패턴 적용 예시  
  ```swift
  class AppServices: Observable { @Dependency(\.apiClient) var apiClient }
  ```
- **Combine과의 하이브리드 패턴**  
  - `.publisher`로 Third-party 라이브러리 연동, `async let` vs `.sink()` 성능 비교  
  - 메모리 누수 예방: `AnyCancellable` 관리, `store(in:)` 활용
- **Computed Property Binding**  
  - `Binding(get:set:)`으로 파생 데이터 양방향 바인딩  
  ```swift
  struct ToggleView: View { @State private var isOn = false; var binding: Binding<String> { Binding(get: { isOn ? "On":"Off" }, set: { isOn = $0 == "On" }) }; var body: some View { TextField("State", text: binding) } }
  ```

#### 3.1.3 고급 레이아웃 및 애니메이션
- **Custom Layout 구현 심화**  
  - 캐시 활용 `cache` 매개변수로 레이아웃 성능 향상  
  - 동적 뷰 크기 변화 처리 팁
- **Grid API & Adaptive Layout**  
  - `LazyVGrid(columns: [GridItem(.adaptive(minimum: 100))])`로 반응형 레이아웃  
- **Physics-based 애니메이션**  
  - `spring(response:dampingFraction:blendDuration:)` 파라미터 튜닝 가이드  
  - `animation(.interactiveSpring(), value: state)` 예시
- **View Transitions**  
  - `.transition(.move(edge: .bottom).combined(with: .opacity))` 복합 전환 효과  

#### 3.1.4 UIKit 통합 및 호환성
- **UIHostingConfiguration**  
  - `UICollectionViewListCell` 등 내부 SwiftUI 삽입 예시  
  ```swift
  cell.contentConfiguration = UIHostingConfiguration { ContentView() }
  ```
- **ScenePhase 동기화**  
  - SwiftUI `@Environment(\.scenePhase)`와 `UIApplicationDelegate` 호환 전략  
- **Adaptive UI 구성**  
  - Size Class 변경 감지: `@Environment(\.horizontalSizeClass)`에 따른 레이아웃 분기

#### 3.1.5 접근성 및 국제화
- **VoiceOver 고도화**  
  - `accessibilityAction(named:_:)`으로 커스텀 제스처 매핑  
  - `AccessibilityFocusState`로 포커스 제어 예시
- **Dynamic Type 및 다국어**  
  - `@ScaledFont` 사용법, 접근성 큰 텍스트 지원 가이드  
  - 번역 시 문자열 포맷 보존: `.formatted(.number.precision(.fractionLength(2)))`

#### 3.1.6 성능 최적화 팁
- **View Identity 관리**  
  - `id(UUID())` 과도 사용 주의, 고유 식별자 활용 가이드  
- **Redraw 최소화**  
  - `EquatableView` 래퍼로 스테이트 불변 시 렌더링 차단  
- **Instruments 활용법**  
  - SwiftUI Rendering Timeline, Memory Graph로 뷰 라이프사이클 분석  



### 3.2 Swift 6.2

#### 3.2.1 고급 동시성 확장
- **`@concurrent` 함수**  
  - 명시적 동시 실행 지시자, 재진입성(reentrancy) 제어 옵션 제공  
  ```swift
  struct MatrixMultiplication {
      @concurrent func multiply(_ a: [[Int]], _ b: [[Int]]) async -> [[Int]] {
          // 동시 분할 작업 수행
      }
  }
  ```
- **Actor 재진입 정책**  
  - `@actorIndependent` 속성으로 안전한 외부 접근  
  - `nonisolated` 메서드에 대한 안전성 보장
- **Structured Concurrency 개선**  
  - `async let`에 `throwing` 병렬 바인딩  
  - `Task.group` 최적화: 취소 및 실패 처리 성능 향상

#### 3.2.2 메모리 및 타입 안전성
- **Inline Arrays**  
  - 고정 크기 배열을 스택에 저장하여 힙 할당 회피  
  - `inline Array<UInt8, 16>` 처럼 선언 가능  
- **`Span<T>`**  
  - 연속 메모리 버퍼 뷰, `withUnsafeSpan` 유틸리티 메서드 제공  
  ```swift
  func processBuffer<T>(data: Span<T>) {
      for element in data { /* 처리 로직 */ }
  }
  ```
- **Move-only 타입 지원**  
  - `@moveOnly` 구조체, 복사 금지 타입 선언으로 안전성 강화

#### 3.2.3 상호운용성 및 빌드 도구
- **C++/Java FFI 개선**  
  - `import CxxModule` 및 `import JavaModule` 키워드로 자동 브리지 생성  
  ```swift
  import CxxWX
  let widget = WXWidget(options: ...)
  ```
- **Swift Containerization**  
  - Docker 컨테이너 내 Swift 컴파일러 실행 지원, `swift package export-docker` 명령어 추가  
- **`swiftly` CLI 및 Build 성능**  
  - `swiftly test --parallel`으로 테스트 병렬 실행  
  - `-Xcompiler -fast-math` 같은 커스텀 플래그 간편 설정  

#### 3.2.4 패키지 매니저 및 ABI 안정성
- **SwiftPM 동적 라이브러리 지원**  
  - `library(type: .dynamic)`로 런타임 로드 가능 모듈 선언  
- **Versioned ABI**  
  - 모듈 간 호환성을 위한 ABI 레이어 강화, 롤링 업데이트 테스트 기능 추가

#### 3.2.5 컴파일러 지시어 및 컨커런시 검증
- **새로운 컴파일러 지시어**
  - `#expressionAs`: 컴파일 시 표현식을 특정 타입으로 캐스팅하도록 강제
    ```swift
    let x = #expressionAs(Int.self, "123")
    ```
  - `#compileTime`: 조건부 컴파일 또는 상수 평가
    ```swift
    let y = #compileTime(1 + 2) // 3
    ```
  - `@backDeployed`: 과거 OS 버전에서도 동작 가능한 함수 선언
    ```swift
    @available(iOS 17, *)
    @backDeployed(before: iOS 16)
    public func doLegacyThing() { ... }
    ```

- **Strict Concurrency Checking 단계**
  - `minimal`: 주요 actor 격리 위반 감지
  - `targeted`: 애플 API 위주 경고
  - `complete`: 모든 비동기 호출 흐름 엄격 검증

- **적용 전략 예시**
  ```swift
  // 예: Xcode 프로젝트 빌드 설정 내에서 적용
  // SWIFT_CONCURRENCY_CHECKS = complete
  ```
  - 초기에는 `targeted` 또는 `minimal`로 시작하고, 점진적 코드 마이그레이션 후 `complete`로 전환 권장
- **SwiftPM 동적 라이브러리 지원**  
  - `library(type: .dynamic)`로 런타임 로드 가능 모듈 선언  
- **Versioned ABI**  
  - 모듈 간 호환성을 위한 ABI 레이어 강화, 롤링 업데이트 테스트 기능 추가



### 3.3 Core ML & Apple Intelligence

> ℹ️ **참고**: 이 섹션에 기술된 일부 기능(예: Apple Intelligence, Writing Tools 등)은 현재 WWDC 2025 기준으로 Apple의 기본 앱 및 시스템 서비스에만 적용되며, **서드파티 개발자는 직접 API를 사용할 수 없습니다**. 퍼블릭 API는 FoundationModels, BNNS, Vision, CreateML 등 일부에 한정되며, Apple Intelligence 기능은 일반 앱에서는 접근 불가입니다.

#### 3.3.1 Foundation Models (AJL)
- **Tool Calling API**  
  - LLM 내부 함수 호출 및 커스텀 작업 플로우 정의 지원  
  ```swift
  import FoundationModels
  let model = try FoundationModel.default()
  let prompt = FMPrompt("Translate to Korean: Hello, world!")
  let result: String = try await model.generate(prompt)
  print(result)
  ```
- **Guided Generation**  
  - 다중 턴 대화, 함수형 응답 구조화  
  - JSON, XML 등 응답 포맷 박싱 예시  
- **Guided Generation**  
  - 다중 턴 대화, 함수형 응답 구조화  
  - JSON, XML 등 응답 포맷 박싱 예시  

#### 3.3.2 BNNS Graph API
- **커스텀 신경망 그래프**  
  - `BNNSGraph` 객체 생성 및 레이어 순서 지정  
  ```swift
  import BNNS
  let graph = BNNSGraph()
  let conv = BNNSFilterCreateConvolutionLayer(graph, /* 설정값 */)
  BNNSGraphCompile(graph)
  BNNSGraphEvaluate(graph)
  ```
- **성능 튜닝**  
  - 하드웨어 최적화: CPU SIMD, 벡터화 옵션  

#### 3.3.3 Vision 프레임워크
- **문서 및 텍스트 인식**  
  - `VNDocumentCameraViewController`로 문서 스캔, OCR 파이프라인 통합  
  ```swift
  let scanner = VNDocumentCameraViewController()
  scanner.delegate = self
  ```
- **얼룩 감지(Smudge Detection)**  
  - `VNDetectHumanHandPoseRequest`, `VNDetectContoursRequest` 조합으로 얼룩 식별  

#### 3.3.4 SpeechAnalyzer
- **실시간 음성 분석**  
  - `SpeechAnalyzer(sessionConfiguration:)`로 음성 신호 강도, 피치, 텍스트 변환 모니터링  
  ```swift
  import Speech
  let analyzer = try SpeechAnalyzer()
  analyzer.delegate = self
  analyzer.startAnalysis()
  ```
- **노이즈 프로파일링**  
  - 잡음 제거 및 Voice Activity Detection(VAD) 지원

#### 3.3.5 Metal ML & MPSGraph
- **텐서 연산 및 모델 실행**  
  - `MPSGraph`로 레이어별 수학 연산 구성  
  ```swift
  import MetalPerformanceShadersGraph
  let graph = MPSGraph()
  let x = graph.placeholder(shape: [1, 224, 224, 3], dataType: .float32)
  let y = graph.convolution2D(x, weights: weightTensor, descriptor: descriptor)
  ```
- **GPU 가속 미세조정**  
  - `GraphTrainingContext`를 통해 온디바이스 fine-tuning 지원

#### 3.3.6 MLX & Create ML Adapter
- **LLM 미세조정 라이브러리(MLX)**  
  - 프롬프트 템플릿, 토크나이저 통합, 하이퍼파라미터 튜닝 기능 제공  
- **Create ML Adapter**  
  - Swift API로 모델 학습 및 내보내기 자동화  
  ```swift
  import CreateML
  let classifier = try MLTextClassifier(trainingData: data)
  try classifier.write(to: URL(fileURLWithPath: "MyClassifier.mlmodel"))
  ```



### 3.4 Metal 4 & 그래픽 기술

#### 3.4.1 Placement Heap & Sparse Resources
- **Placement Heap**  
  - 메모리 풀에서 텍스처 및 버퍼를 동적으로 할당 및 해제  
  - CPU/GPU 동기화 오버헤드 감소 팁  
- **Sparse Resource Binding**  
  - 거대한 애셋을 작은 타일 단위로 로드, 메모리 사용량 최적화  
  ```swift
  let heapDescriptor = MTLHeapDescriptor()
  heapDescriptor.size = 64 * 1024 * 1024
  let heap = device.makeHeap(descriptor: heapDescriptor)!
  let textureDesc = MTLTextureDescriptor.texture2DDescriptor(pixelFormat: .rgba8Unorm, width: 4096, height: 4096, mipmapped: false)
  textureDesc.storageMode = .private
  textureDesc.heap = heap
  let texture = heap.makeTexture(descriptor: textureDesc)!
  ```

#### 3.4.2 Dedicated Compiler Queue & Pipeline Object Harvesting
- **Dedicated Compiler Queue**  
  - 런타임 셰이더 컴파일 오버헤드를 분리하여 프레임 레이팅 안정화  
- **Pipeline Object Harvesting**  
  - 앱 실행 시 사용된 파이프라인 상태 객체를 자동 수집, 최적화된 번들 생성  
  ```swift
  let library = try device.makeLibrary(source: shaderSource, options: nil)
  let pipelineDesc = MTLRenderPipelineDescriptor()
  pipelineDesc.vertexFunction = library.makeFunction(name: "vertex_main")
  pipelineDesc.fragmentFunction = library.makeFunction(name: "fragment_main")
  let pipelineState = try device.makeRenderPipelineState(descriptor: pipelineDesc)
  ```

#### 3.4.3 MetalFX & GPU 가속 FX
- **Upscale & Denoise**  
  - `MTLFXUpscaler`와 `MTLFXDenoiser` 객체로 초고해상도 및 노이즈 제거 처리  
  ```swift
  let upscaler = device.makeFXUpscaler(descriptor: upscalerDescriptor, sourceTexture: input, destinationTexture: output)
  upscaler?.encode(commandBuffer: cb)
  ```
- **효율적 메모리 관리**  
  - 중간 버퍼 재사용, 커맨드 버퍼 풀링 패턴 소개

#### 3.4.4 레이트레이싱 & 가속 구조
- **레이트레이싱 인터섹션 Shader**  
  - `intersection_function` 정의 및 `MTLAccelerationStructureDescriptor` 설정  
- **가속 구조 빌드 최적화**  
  - `MTLAccelerationStructureControlOptions`로 빌드 모드 선택 (fast build vs fast trace)

#### 3.4.5 디버깅 및 프로파일링 도구
- **Metal System Trace**  
  - Instruments에서 GPU 캡처, 타임라인 분석 지원  
- **Shader Profiler**  
  - 런타임 셰이더 성능, 사이클 카운트, 레지스터 사용량 조회



### 3.5 ARKit

#### 3.5.1 RoomPlan & Anchors
- **Room Anchor (RoomPlan)**  
  실내 공간의 벽, 바닥, 천장을 자동 감지하고 앵커 생성  
  ```swift
  import RoomPlan
  let config = RoomPlan.Configuration()
  session.run(config)
  ```
- **Custom Anchor 처리**  
  `ARExperienceAnchor`를 사용해 앱 고유의 앵커 메타데이터 관리  

#### 3.5.2 Plane Detection & Tracking
- **Slanted Plane Detection**  
  기울어진 표면 인식(탁자 모서리, 비스듬한 벽 등)  
- **TrackingState API**  
  `scene.session.currentFrame?.camera.trackingState`로 실시간 상태 확인  

#### 3.5.3 Object & Face Tracking
- **Reference Object Tracking**  
  `ARReferenceObject` 로드 후 지속 추적  
  ```swift
  let object = try Entity.load(named: "MyARObject")
  let anchor = try ARObjectAnchor(referenceObject: referenceObject, transform: matrix)
  ```
- **Face Tracking & BlendShapes**  
  `ARFaceAnchor`를 통해 얼굴 표정, BlendShape 수치 확인 및 `FaceMesh` 시각화  

---

### 3.6 RealityKit

#### 3.6.1 SwiftUI 연동 & Entity Lifecycle
- **ViewAttachmentComponent**  
  SwiftUI 뷰를 RealityKit 엔티티에 부착  
  ```swift
  let viewEntity = ModelEntity()
  viewEntity.components[ViewAttachmentComponent.self] = .init(rootView: ContentView())
  ```
- **Entity Lifecycle 관리**  
  `SceneEvents.Update`로 매 프레임 애니메이션, 상태 동기화 처리  

#### 3.6.2 Mesh & Rendering 기능
- **MeshInstancesComponent**  
  대량 인스턴스 렌더링 최적화  
- **OcclusionMaterial**  
  실제 환경 오클루전 처리로 현실감 강화  

---

### 3.7 SceneKit

#### 3.7.1 SceneExporter & RealityKit 마이그레이션
- **SceneExporter 유틸리티**  
  `SCNScene`을 RealityKit `.reality` 파일로 내보내기  
  ```swift
  import SceneKit
  let scene = try SCNScene(named: "scene.scn")
  SceneExporter.export(scene, to: URL(fileURLWithPath: "scene.reality"))
  ```

#### 3.7.2 재질(Material) 매핑
- SceneKit `SCNMaterial` ↔ RealityKit `Material` 속성 자동 변환 규칙 소개  

---

### 3.8 네트워킹 & 통신

네트워킹 스택과 통신 프레임워크의 최신 개선 사항을 정리합니다.

#### 3.8.1 HTTP/3 & 네트워크 성능
- **HTTP/3 기본 지원**  
  - QUIC 프로토콜 기반의 다중 경로 및 헤더 압축 제공  
  - `URLSessionConfiguration`에서 `allowsHTTP3` 옵션 활성화 예시  
  ```swift
  let config = URLSessionConfiguration.default
  config.allowsHTTP3 = true
  let session = URLSession(configuration: config)
  ```
- **네이티브 Instruments 트레이스**  
  - HTTP/3 연결, 스트림 상태, 패킷 전송 분석 도구 지원  

#### 3.8.2 Network.framework & Wi‑Fi Aware
- **Network.framework async/await**  
  - `NWConnection` 및 `NWListener`의 비동기 API  
  ```swift
  let conn = NWConnection(host: "example.com", port: 443, using: .tls)
  await conn.start()
  let data = try await conn.receive(minimumIncompleteLength: 1, maximumLength: 65536)
  ```
- **Wi‑Fi Aware (NAN)**  
  - 근거리 디바이스 간 직접 통신 지원, `NANSession` API 소개

#### 3.8.3 Background Assets & URLSession
- **BackgroundAssets 다운로드/업로드**  
  - `URLSession`의 `background` 세션에서 대용량 파일 안정적 전송  
  ```swift
  let backgroundConfig = URLSessionConfiguration.background(withIdentifier: "com.example.bg")
  let backgroundSession = URLSession(configuration: backgroundConfig, delegate: self, delegateQueue: nil)
  ```
- **재개(resume) 및 취소 처리**  
  - `downloadTask(withResumeData:)`, `cancel(byProducingResumeData:)` 패턴 설명

#### 3.8.4 실시간 통신 & WebSocket
- **URLSessionWebSocketTask**  
  - `send(_:completionHandler:)`, `receive(completionHandler:)` API 예시  
  ```swift
  let webSocket = session.webSocketTask(with: URL(string: "wss://echo.websocket.org")!)
  webSocket.resume()
  webSocket.send(.string("Hello")) { error in /* ... */ }
  ```
- **서버 푸시 알림 & Server Push**  
  - HTTP/2 `push` 이벤트, `URLSession.delegate` 내 `didReceive challenge` 활용

#### 3.8.5 보안 및 인증
- **TLS 1.3 최적화**  
  - `NWProtocolTLS.Options`로 커스텀 암호화 스위트 설정  
- **OAuth2 및 인증 흐름**  
  - `ASWebAuthenticationSession`을 이용한 OAuth2 로그인 예시  
  ```swift
  let authSession = ASWebAuthenticationSession(url: authURL, callbackURLScheme: callbackScheme) { callbackURL, error in /* ... */ }
  authSession.start()
  ```


## 4. 부록 및 출처

### WWDC 2025 주요 세션 목록 및 번호

| 플랫폼 / 주제               | 세션명 및 번호                                     |
|----------------------------|----------------------------------------------------|
| Foundation Models           | Meet the Foundation Models (WWDC25-286)            |
| App Intents 심화            | Explore new advances in App Intents (WWDC25-275)   |
| SwiftUI 업데이트            | What’s new in SwiftUI (WWDC25-256)                 |
| Swift 동시성               | Embracing Swift concurrency (WWDC25-250)           |
| Metal 4 소개                | What’s new in Metal (WWDC25-262)                   |
| SwiftUI에서의 리치 텍스트   | Cook up a rich text experience (WWDC25-243)        |

> 📝 **출처**: 본 문서는 [Apple WWDC 2025](https://developer.apple.com/wwdc25/)에서 발표된 공식 콘텐츠 및 개발자 세션 영상, 문서를 기반으로 작성되었습니다.
