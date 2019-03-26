
# AR Face Tracking Use ARKit

Face Tracking을 위해 TrueDepth 전면 카메라를 지원하는 iPhone이 필요하다.(iPhoneX 이상)


## 시작하기

- Xcode를 실행하고 Single View App 템플릿을 기반으로 새로운 프로젝트를 만든다.
- `import ARKit` 를 추가한다.
- Main.stroyboard 에서 ViewController의 최상위 view를 선택한 상태에서 클래스를 `ARSCNView`로 변경합니다.

<img width="1399" alt="image" src="https://user-images.githubusercontent.com/33486820/55002731-73949580-501a-11e9-987a-1598e94263d2.png">

[`ARSCNView`](https://developer.apple.com/documentation/arkit/arscnview)는 `SceneKit` 컨텐츠를 사용하여 증강 현실 경험을 보요주는 view이다. 카메라 피드를 표시하고 `SCNNode`를 표시해준다.

- ViewController에서 ARSCNView로 설정한 최상위 view를 sceneView라는 IBOutlet 으로 연결해준다.


```swift
override func viewWillAppear(_ animated: Bool) {
  super.viewWillAppear(animated)
        
  // 1
  let configuration = ARFaceTrackingConfiguration()
        
  // 2
  sceneView.session.run(configuration)
}
    
override func viewWillDisappear(_ animated: Bool) {
  super.viewWillDisappear(animated)
        
  // 1
  sceneView.session.pause()
}
```

- `viewWillAppear`
	- 1. 얼굴을 추적하는 **configuration**을 생성한다
	- 2. ARSCNView의 기본 ARSession 속성을 사용하여 얼굴 추적 구성 실행한다.

- `viewWillDisappear`
	- 1. ARSession을 pause한다
    
    
### TrueDepth 를 지원 하는 디바이스에서만 사용가능

사용자에게 TrueDepth가 전면 카메라에 지원되는 디바이스 인지 아닌지에 따라 피드백을 해야한다.


```swift
guard ARFaceTrackingConfiguration.isSupported else {
  fatalError("이 장치에서는 얼굴 추적 기능이 지원 되지 않습니다.")
}
```


### Face Anchors and Geometries

- [`ARFaceAnchor`](https://developer.apple.com/documentation/arkit/arfaceanchor)

Face Tracking AR Session에서 감지된 얼굴의 포즈, 위상 및 표현에 대한 정보

```swift
class ARFaceAnchor : ARAnchor
```

얼굴 추적 ARSession을 실행할 때([`ARFaceTrackingConfiguration`](https://developer.apple.com/documentation/arkit/arfacetrackingconfiguration) 참고), 세션은 전면 카메라로 사용자의 얼굴을 탐지했을 때 ARFaceAnchor 객체의 앵커 목록에 자동으로 추가한다. 각 face anchor는 현재 위치와 방향, topology(위상) 및 facial expression(표정)에 대한 정보를 제공한다.

#### Tracking Face Position and Orientation(얼굴 위치 및 방향 추적)

상속된 [`transform`](https://developer.apple.com/documentation/arkit/aranchor/2867981-transform) 속성은 실세계 좌표에서 세션 구성의 [`worldAlignment`](https://developer.apple.com/documentation/arkit/arconfiguration/2923550-worldalignment) 속성에 의해 지정된 것과 관련된 좌표 공간에서. 얼굴의 현재 위치와 방향을 설명한다. 이 변환 매트릭스를 사용하여 AR 씬(scene)의 얼굴에 "attach"할 가상 컨텐츠를 배치해야한다.

<img width="613" alt="image" src="https://user-images.githubusercontent.com/33486820/55006077-7c886580-5020-11e9-8664-856239bb0940.png">


> 좌표계는 오른손 기준

- 양(+)의 x 방향은 관점자(viewer)의 오른쪽( 즉, 얼굴 기준의 왼쪽)
- 양(+)의 y 방향은 위로 향하고(실세계가 아닌 얼굴 자체에 입장에서 즉 관찰자 기준이 아닌 대상의 얼굴의 위쪽으로)
- 양(+)의 z 방향은 얼굴에서 (viewer 쪽으로) 바깥쪽으로 향한다.
 
 
#### Using Face Topology(얼굴의 위상 사용)


이 [`geometry`](https://developer.apple.com/documentation/arkit/arfaceanchor/2928271-geometry) 속성은 얼굴의 자세한 topology(위상)을 나타내는 [`ARFaceGeometry`](https://developer.apple.com/documentation/arkit/arfacegeometry)객체를 제공한다. 이 객체는 감지 된 얼굴의 크기, 모양 및 현재 식과 일치하도록 일반 모델을 준수한다.

예를 들어 가상 메이크업이나 문신을 바르는 등 사용자의 얼굴 모양을 따르는 콘텐츠를 오버레이하는 기준으로 이 모델을 사용할 수 있다. 또한 이 모델을 사용하여 눈에 보이는 콘텐츠를 렌더링하지 않지만(카메라 이미지가 투과되도록 허용) 장면의 다른 가상 콘텐츠에 대한 카메라의 시야를 방해하는 3D 모델인 폐색 지오메트리를 만들 수 있다.


#### Tracking Facial Expressions(얼굴 감정 추적)


[`blendShapes`](https://developer.apple.com/documentation/arkit/arfaceanchor/2928251-blendshapes) property(혼합형 쉐이프 특성) 은 현재 얼굴 표정의 높은 수준의 모델을 제공하며, 중립 구성에 대한 특정 얼굴 형상의 움직임을 나타내는 일련의 명명된 계수를 통해 설명된다. 혼합형 모양 계수를 사용하여 사용자의 얼굴 표정을 따르는 방법으로 캐릭터나 아바타와 같은 2D 또는 3D 콘텐츠를 애니메이션할 수 있다.


- [`ARFaceGeometry`](https://developer.apple.com/documentation/arkit/arfacegeometry)
- [`ARSCNFaceGeometry`](https://developer.apple.com/documentation/arkit/arscnfacegeometry)






