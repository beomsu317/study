# Why Do We Test Our Code?

## Testing in Android

* 테스트 케이스는 코드가 동작하는 것을 테스트하기 위함이다.
* 테스트 없이는 수동으로 동일한 기능에 대해 여러번 검증해야 한다. 이는 시간을 많이 소비하게 한다.
* JUnit은 원할 때 클릭 한 번으로 코드에 대한 테스트를 수행하게 해준다.

### Unit Tests

* 유닛 테스트는 앱의 단일 컴포넌트만 테스트한다.
* 테스트 속도가 빠르다.
* 유닛 테스트는 앱의 다른 컴포넌트에 의존하지 않는다.
* 앱의 70% 정도의 테스트 케이스가 만들어져야 한다. 

### Integration Tests

* 앱의 다른 컴포넌트들이 어떻게 함께 동작하는지를 검증한다. 
  * 예를 들어, 프레그먼트가 뷰모델과 상호작용 하는 것
* Integraion Tests와 integrated tests를 혼동하기 쉽다. 
  * Integrated tests는 안드로이드 컴포넌트에 의존한 테스트를 의미한다.
* 앱의 20% 정도의 테스트 케이스가 만들어져야 한다.

### UI Tests

* End-to-end tests라 불리기도 한다.
* 나머지 앱의 10% 정도의 테스트 케이스가 만들어져야 한다.
* UI 테스트는 우리가 예상한 특정 상태를 가지고 있는지를 검증한다.
  * Checkbox가 체크되었는지, RecyclerView에 지정된 아이템이 있는지 등
* 항상 emulator에서 수행된다. 

## References 
* [Why Do We Test Our Code? - Android Testing - Part 1](https://www.youtube.com/watch?v=EkfVL5vCDmo&list=PLQkwcJG4YTCSYJ13G4kVIJ10X5zisB2Lq&index=1)