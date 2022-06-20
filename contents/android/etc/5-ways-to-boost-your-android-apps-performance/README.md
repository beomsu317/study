# **5 Ways to Boost Your Android App's Performance**

안드로이드 앱 퍼포먼스를 늘릴 수 있는 방법 5가지를 알아보자.

## Use r8

많은 사람들이 r8은 프로가드라고 알고 있지만, 프로가드와 유사한 코드를 최적화하기 위한 툴이다. apk를 작게 만들어줄 뿐만 아니라 리버스 엔지니어링도 어렵게 만든다. 여기선 앱을 작게 만드는 것에 집중한다.

앱을 작게 만들기 위해 r8은 3가지를 수행한다.

1. 모든 클래스, 함수, 필드를 알아볼 수 없는 짧은 이름으로 변경한다. 이름이 짧아지므로 앱도 작아지게 된다.
    - 리버스 엔지니어링도 어렵게 만드는 효과도 얻는다.
2. 사용되지 않는 클래스나 함수를 제거해준다. 만약 용량이 큰 라이브러리를 포함했고 2개의 함수만을 사용하면 앱에서 필요하지 않은 부분들을 버리고 실제 사용하는 2개의 함수만을 취한다.
3. 사용되지 않는 리소스를 제거해준다.

r8은 자동적으로 활성화된다. 다음과 같이 설정하면 적용된다.

```groovy
buildTypes {
		release {
				minifyEnabled true
				shrinkResources true
				// ... 
		}
}
```

## Use the android studio profiler

네트워크, 메모리 소비량 등 앱에서 사용되는 모든 자원에 대해 보여준다. 특정 기능에 대한 테스트를 수행하며 얼마나 자원이 사용되는지 확인할 수 있다.

## Cache data whenever possible

가능하면 데이터를 캐싱해라. 네트워크로부터 받은 데이터를 재사용하면 캐싱해라. 받아온 데이터를 Room DB, 파일로 저장하거나 coil, glides, picasso와 같은 이미지 캐싱 라이브러리를 사용해라.

## Optimizes your app’s memory consumption

메모리 릭을 체크해라. 메모리 릭은 가비지 컬렉터가 collect 하지 않을 때 발생하는데, Leak Canary를 통해 메모리릭을 탐지할 수 있다.

## Optimize Networking

네트워크 사용량을 최적화해라. 네트워킹은 폰의 배터리를 닳게 하는 주 원인이므로 최적화가 필요하다. 단말기는 라디오 칩을 가지고 있고, 이를 통해 cell phone tower와 통신한다. 앱이 요청을 생성할 때 단말기는 이 라디오 칩을 활성화 시켜 cell phone tower와 통신하며 응답을 받기 위해 조금 길게 활성화 되어 있는다. 기본적으로 라디오 칩은 비활성화 된 상태이다. 요청이 많을수록 라디오 칩은 응답을 받기 위해 오래 활성화되어 있게 된다. 따라서 여러개의 요청을 보내는 것보다 하나의 큰 요청을 보내는 것이 좋다.

## References

* [5 Ways to Boost Your Android App's Performance](https://www.youtube.com/watch?v=epkAPnF5qrk)