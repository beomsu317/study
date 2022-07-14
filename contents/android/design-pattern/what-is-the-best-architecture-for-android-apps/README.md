# What Is the Best Architecture for Android Apps?

앱에서 어떤 디자인 패턴을 사용해야 하는지 알아보자. MVVM, MVI, MVP, MVC 등. MVVM, MVI를 주로 사용한다.

* M : Model
    * data source that you use in your app
        * local database
        * remote api
* V : View
    * represent the UI layer of your app
        * fragment, activity, composables

## MVVM

* Model-View-ViewModel
* VM에는 앱의 비즈니스 로직을 구현한다. 비즈니스 로직을 분리하기 위해 선택적으로 domain 레이어를 구현하기도 한다.
* VM은 data 레이어와 상호작용 한다. data 레이어에서 반환된 값(local database나 remote API)을 state로 매핑한다. 즉, UI 레이어에 UI state를 제공하여 observe
  하도록 한다.
* 각 state는 독립적이다.
* VM은 가끔 많은 state를 포함한다.
* `SavedStateHandle`을 이용해 프로세스가 다시 열리더라도 기존 상태를 유지할 수 있다.

## MVI

* Model-View-Intent
* 안드로이드 시스템의 `intent`와 관련이 없으며, 사용자의 의도를 의미한다.
* UI 이벤트를 UI에서 VM으로 전달한다. VM은 이 이벤트를 전달받아 처리한다.
    * sealed class를 통해 구현
* 하나의 스크린에 하나의 state가 존재한다. `copy()`를 통해 상태를 업데이트한다.
    * 이는 race condition에 취약하다. `StateFlow`의 경우 thread-safe 하여 race condition이 발생하지 않지만 `copy()`는 thread-safe 하지 않다. 즉,
      race condition이 발생할 수 있다.
* 프로세스가 죽은 후 처리가 어렵다. `SaveStateHandle`을 통해 모든 screen state를 복구할 수 있지만, 모든 state를 복구하기 원하지 않을 때가 있다. 
  * 로딩이 진행중일 때 프로세스가 종료된 후 재시작되었을 때 로딩 UI가 무한정 실행될 수 있다. 
* 어느 screen state가 업데이트되더라도 전체 UI가 리렌더링 된다. 하나의 state에 의존하기 때문이다.

## MVP

* Model-View-Presenter
* 프레젠터는 view를 알고 view는 프레젠터를 알고 있다.

## MVC

* Model-View-Controller
* Controller는 view를 업데이트할 책임이 없다.

## References

* [What Is the Best Architecture for Android Apps?](https://www.youtube.com/watch?v=cnU2zMnmmpg&t=29s)