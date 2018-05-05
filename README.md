## [iOS] AVFoundation Programming Guide

[AVFoundation Programming Guide](https://developer.apple.com/library/content/documentation/AudioVideo/Conceptual/AVFoundationPG/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40010188-CH1-SW3)을 보면서 공부한 내용을 제가 이해한 것을 바탕으로 정리한 내용입니다. 올바르지 않은 정보가 있다면 피드백 부탁드리겠습니다. 

> 몇몇 단어와 용어는 영어 단어를 통해 이해하는 것이 더욱 직관적이므로 굳이 한글로 바꾸어 설명하지는 않았습니다. 이점 참고하여 읽어주시기 바랍니다.

---

###About AVFoundation

AVFoundation은 time-based audiovisual 미디어를 생성하는데 사용할 수 있는 프레임워크들 중 하나입니다. AVFoundation은 Objective-C 기반의 인터페이스도 제공하여 보다 디테일한 수준에서의 작업도 가능케합니다.

디테일한 수준의 작업이라하면 미디어 파일을 검사, 작성, 편집 또는 재인코딩 같은 것들이 있습니다. 또한 실시간으로 재생 중에 디바이스로부터 입력 스트림을 받아와 비디오를 조작할 수도 있습니다. 

> 디테일한 수준의 작업이라고 소개 된 부분인데 이 부분에 대해서는 아직은 경험이 부족한 탓인지 와닿지는 않네요. 추후에 설명을 보강하도록 하겠습니다. 

AVKit은 iOS와 OSX에서 각각 UIKit과 AppKit 위에 위치하고 AVFoundation은 UIKit과 AppKit 아래에 위치합니다. 그 아래에는 Corea Audio, Core Media, Core Animation 등이 위치합니다.

|      AVKit       |      AVKit       |
| :--------------: | :--------------: |
|    **UIKit**     |    **AppKit**    |
| **AVFoundation** | **AVFoundation** |

고수준을 요구하는 작업이 아닌 일반적인 미디어 관련 작업에 대해서는 상위 수준의 프레임워크를 사용해야 합니다. 즉 단순히 영상을 재생하고 싶다면 AVKit 프레임워크를 사용하면 됩니다!

---

### At a Glance

AVFoundation 프레임워크는 두 가지 종류의 API를 제공합니다. 한 가지는 비디오 재생에 사용되는 API고 한 가지는 오직 오디오 재생에만 사용되는 API입니다. 후자는 오디오를 보다 쉽게 다루는 서비스를 제공합니다.

- 사운드 파일을 재생하려면 `AVAudioPlayer` 를 사용할 수 있습니다.
- 오디오를 녹화하려면 `AVAudioRecorder` 를 사용할 수 있습니다.

또한 `AVAudioSession` 을 사용하여 오디오의 전반적인 행위에 대한 것을 구성할 수 있습니다.

---

### Asset 사용하기

Asset은 파일이나 사용자의 포토 라이브러리 같은 저장소에 위치한 미디어로부터 가져올 수 있습니다. 예를 들어 Movie Asset을 갖고 있다면 이로부터 이미지를 추출할 수 있으며 다른 포맷으로 변환할 수 있고 컨텐츠를 다듬을 수도 있습니다. 하지만 Asset 객체를 만들었다고 해서 그 즉시 해당 Asset에 대한 모든 정보를 가져올 수는 없습니다. 

#### Asset 객체 생성 (Creating and Asset Object)

사용하려는 Asset을 리소스의 URL로 식별할 수 있다면 다음과 같이 `AVURLAsset` 생성자를 이용해서 Asset 객체를 생성할 수 있습니다. 즉 URL로 Asset을 생성할 수 있다는 것입니다.

```swift
let url = URL(string: "A URL that identifies an audiovisual asset such as a movie file")
let asset = AVURLAsset(url: url, options: nil)
```

#### Option과 함께 Asset 객체 생성 및 초기화 (Options for Initializing an Asset)

`AVURLAsset` 생성자는 두 번째 매개변수로 Key-Value 형태의 딕셔너리 타입 옵션 값을 받습니다. 이 옵션 값에 해당하는 딕셔너리로 `AVAsset` 객체를 생성하는데 여러 옵션을 부여할 수 있습니다. 

이러한 옵션들을 통해 `AVAsset` 객체를 생성하면 생성된 직후 바로 특정한 임의의 시간에 대해 디테일한 접근이 가능합니다. 또한 정확한 총 재생 시간 정보와 함께 `AVAsset` 객체를 생성하는 것도 가능합니다. 하지만 이렇게 객체를 생성한 직후 시간을 바탕으로 하는 정교한 작업을 가능케 하기 위해서는 객체가 생성되는 과정에 상당한 오버헤드를 발생시키는 수행 작업이 발생합니다. 그렇기 때문에 일반적으론 대략적인 재생 시간을 사용하는 것이 보다 덜 부담스러운 작업이며 적합한 방법입니다. 

```swift
let url = URL(string: "A URL that identifies an audiovisual asset such as a movie file")
let options = [AVURLAssetPreferPreciseDurationAndTimingKey:false]
let asset = AVURLAsset(url: url, options: options)
```

### Asset 사용 준비 (Preparing an Asset for Use)

Asset을 생성했다고 해당 아이템에 모든 정보에 대해 그 즉시 접근할 수 있다는 걸 의미하는 것은 아닙니다. 이렇게 생성된 직후 아직 채워지지 않은 값을 받아오기 위한 상황에서 현재 스레드를 멈추어 동기적으로 해당 값이 채워질 때까지 기다려 값을 가져오는 것보단 Block에 정의한 Completion handler를 통해 응답을 비동기적으로 기다리는 방법이 더욱 적합합니다.

이러한 Block에는 요청에 의해 돌아온 응답에 담긴 데이터의 상태에 따른 코드를 작성해주시면 됩니다. 또한 원격에 존재하는 데이터를 받아오는 과정에서의 네트워크 장애나 데이터를 불러오는 행위가 최소됨에 따라 발생하는 상황에도 항상 준비가 되어있어야 합니다.

```swift
// URL of a bundle asset called 'example.mp4'
let url = Bundle.main.url(forResource: "example", withExtension: "mp4")!
let asset = AVAsset(url: url)
let playableKey = "playable"

// Load the "playable" property
asset.loadValuesAsynchronously(forKeys: [playableKey]) {
    var error: NSError? = nil
    let status = asset.statusOfValue(forKey: playableKey, error: &error)
    switch status {
    case .loaded:
		// 성공적으로 데이터를 받아왔을 경우
    case .failed:
        // 실패했을 경우
    case .cancelled:
        // 도중에 취소된, 중단된 경우
    default:
        // 기타 다른 경우
    }
}                
```

### 비디오로부터 이미지 가져오기 (Getting Still Images From a Video)

비디오 `AVAsset` 객체에서 썸네일과 같은 이미지를 얻기 위해서는 `AVAssetImageGenerator` 객체를 사용합니다. 이렇게 `AVAsset` 객체가 생성될 때 이미지를 가져올 수 있지만 생성되는 시점에 이러한 시각적인 요소가 존재하지 않을 수도 있기 때문에 이런 이미지를 불러오기 전에 생성되는 시점에 시각적인 요소가 존재하는지에 대한 검사가 선행되어야 합니다.

```swift
if asset.tracks(withMediaType: video).count > 0 {
    // 비디오로써 시각적인 요소가 존재한다는 의미.
    // 썸네일을 추출해낼 수 있다.
}
```

#### 하나의 이미지 가져오기 (Generating a Single Image)

`copyCGImage(at:actualTime:)` 메소드를 이용하여 비디오의 특정 시간대에 해당하는 이미지를 추출할 수 있습니다. 이 메소드는 예외가 발생할 수 있으니 반드시 예외 처리 코드에서 작성해주어야 합니다.

#### 여러개의 연속된 이미지 가져오기 (Generating a Sequence of Images)

여러 개의 이미지를 생성하려면 `AVAssetImageGenerator` 객체의`generateCGImagesAsynchronously(forTimes:completionHandler:)` 메소드를 사용해야 합니다. 첫 번째 매개변수로는 원하는 이미지들에 해당하는 시간들을 명시해주고 두 번째 매개변수는 Callback Block으로 이미지 생성에 대한 요청이 완료되었을 때 실행되는 것입니다. 이렇게 Block을 구현해줄 때 Block 내부에서 요청에 대한 결과를 확인하여 이미지가 정상적으로 생성되었는지 아닌지를 확인할 수 있습니다. 

그리고 이 Block은 첫 번째 인자로 넘어오는 각각의 시간들의 이미지를 추출할 때마다 한번 씩 호출이 됩니다.

### 비디오 다듬고 변환하기 (Trimming and Transcoding a Movie)

`AVAssetExportSession` 객체를 이용하여 Asset(비디오)의 포맷을 다른 포맷으로 변환시킬 수 있으며 해당 Asset을을 다듬을 수도 있습니다. `AVAssetExportSession` 객체는 이러한 변환 과정을 제어하는 객체로 비동기적으로 해당 Asset을 변환합니다. 그리고 변환된 객체를 반환합니다. 변환할 Asset과 변환에 사용할 옵션들과 함께  `AVAssetExportSession` 객체를 초기화합니다. 이렇게 생성된 객체로 출력 결과물의 URL과 파일의 타입들을 명시해주고 메타데이터와 네트워크 사용에 맞게 최적화되어야 하는지에 대한 설정을 선택적으로 지정할 수 있습니다.

```swift
let exporter = AVAssetExportSession(asset: asset, presetName: AVAssetExportPresetHighestQuality) //변환활 Asset과 변환에 대한 옵션을 지정

let filename = "filename.m4v" //변환될 파일의 이름
let documentsDirectory = FileManager.default.urls(for: FileManager.SearchPathDirectory.documentDirectory, in: FileManager.SearchPathDomainMask.userDomainMask).last! // 변환될 파일의 저장 경로 생성
let outputURL = documentsDirectory.appendingPathComponent(filename) // 경로 + 파일 이름
exporter?.outputURL = outputURL // 결과물 경로 지정
exporter?.outputFileType = AVFileTypeAppleM4V // 결과물 파일 타입 지정
exporter?.exportAsynchronously(completionHandler: {

})
```

위의 completionHandler는 이러한 변환 작업이 완료되면 호출됩니다. 그러므로 핸들러 구현 코드에는 변환 결과가 성공했는지, 실패했는지 혹은 취소되었는지와 같은 상태를 검사해야 합니다.

```swift
exporter?.exportAsynchronously(completionHandler: {
    switch exporter?.status() {
        case .failed:
        	print("Export failed: \(exportSession.error().localizedDescription)")
        case .cancelled:
        	print("Export canceled")
        default:
        	break
    }
})
```

또한 위의 `.cancelled` 에 해당하는 중도 취소 작업을  `AVAssetExportSession` 의 `cancelExport` 이용하여 직접  호출할 수도 있습니다. `.fail`은 이러한 변환 과정에서 이미 존재하는 파일에 덮어쓰기를 시도하거나 어플리케이션의 샌드박스 외부에 파일을 저장하려 할 때 발생합니다. 또한 도중에 전화가 오거나 현재 변환 과정이 진행 중인 어플리케이션이 백그라운드로 들어갈 때 그리고 다른 어플리케이션이 재생을 시작했을 때는 `.fail`을 결과로 반환합니다. 

이런 상황에서는 반드시 변환의 실패를 사용자에게 알려야 하고 사용자로 하여금 다시 변환 과정을 시작할 수 있게끔 해야 합니다.

