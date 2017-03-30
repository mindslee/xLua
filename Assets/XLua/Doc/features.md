# Features

## 목록

* Lua 가상머신의 지원
  * Lua5.3.3
  * Luajit2.1beta2
* Unity3D 버전
  * Unity5
  * Unity4
* 플랫폼 지원
  * windows 64/32
  * android
  * ios 64/32/bitcode
  * osx
* Lua와 C#간의 접근
  * 바인딩 코드 생성
  * 리플렉션
* 편의성
  * 압축풀어 바로 사용가능
  * 바인딩 코드를 자동으로 생성해 준다.
  * 바인딩 코드 방식과 리플렉션 방식을 자유롭게 선택 사용할 수 있다.
  * GC없는 API를 쉽게 사용할 수 있다.
  * 메뉴를 통해 간단하게 이해하고 사용할 수 있음.
  * 여러방식으로 설정할수 있으며, 모듈에 따라 분리가능, 대상 클래스에 직접 Attribute 표기를 할수 있음.
  * link.xml을 자동생성하여 코드변조를 차단함.
  * Plugins에 cmake컴파일을 채용하여 더 간단해짐
  * 핵심코드를 제외하여 바인딩 코드생성을 할 수 있으며, 생성된 목록을 언제든지 삭제할 수 있음.
* 성능
  * Lazyload기술을 사용하여 불필요한 코스트를 줄인다.
  * lua함수는 C#의 delegate에, lua table을 C#의 interface에 매핑하여 C#의 gc alloc 비용없이 사용을 할 수 있다.
  * 모든 기본 타입, 모든 열거형, 모든 필드가 value 타입인 struct를 Lua와 C#간의 C# gc alloc 없이 전달하여 사용할 수 있다.
  * LuaTable，LuaFunction이라는 GC없는 인터페이스를 제공함.
  * 바인딩 코드 생성시 static analyze를 통해 더욱 최적화된 코드를 생성함.
  * C#과 Lua간의 포인터 전달 가능
  * 이미 파괴된 UnityEngine.Object의 레퍼런스를 자동으로 삭제해줌.
* 확장성
  * 코드수정없이 Lua 서드파티 패키지 사용가능
  * 코드 생성 엔진에서 2차개발을 위한 interface를 제공함.
 
## 어떤 C# 구현에 대해 HotFix를 지원하는가

  * constructor
  * destructor
  * member 함수
  * static function
  * generic function
  * operator overload
  * property
  * static property
  * event
 
## Lua 코드 로딩

* string 로드
  * 로드후 직시 실행 지원
  * 로드후 delegate나 LuaFunction으로 리턴할 수 있으며, delegate나 LuaFunction사용시 인자를 줄수 있음.
* Resources디렉토리의 파일
  * 그냥 require하면 됨.
* Custom Loader
  * Lua에서 require시 사용됨
  * require시 loader에 인자를 전달하고, loader는 로드한 lua코드를 리턴함
* Lua 기본 방식
 * Lua 기본 방식 모두 사용가능
 
## Lua에서 C#사용

* C# 객체의 생성
* C# static property, field의 접근
* C# static method
* C# member propety, field
* C# member method
* C# 클래스 상속
  * child class의 객체에서 직접 parent class의 method와 property에 접근가능
  * child class 모듈은 직접 parent class의 static method와 property에 접근가능
* Extension methods
  * 보통의 member method와 동일하게 사용
* 인자의 in/out 속성（out，ref）
  * out는 lua의 리턴값 하나에 대응
  * ref는 lua의 인자 하나, 리턴값 하나에 대응함.
* 함수 오버로딩
  * 오버로딩 지원
  * lua의 데이터타입이 C#에 비해 작기 때문에, 판단이 안되는 상황이 있을수 있음. extension method통해 사용할 수 있음.
* 오퍼레이터 오버로딩
  * 지원하는 오퍼레이터 : +，-，*，/，==，-，<，<=， %，[]
  * 기타 오퍼레이터는 extension method로 사용
* default parameter
  * C#의 파라미터는 기본값이 있으며, lua에서 사용가능
* 가변인자
  * 가변인자에 대응해서 하나씩 입력하면됨. 어레이화 할 필요없음
* 제네릭 메소드
  * static method는 자동으로 바인딩되어 사용가능
  * member method는 extension method를 통해 추상화되어 사용
* 열거형
  * 숫자나 문자열형태에서 열거형으로 전환 지원
* delegate
  * C# delegate의 사용
  * + operator
  * - operator
  * lua의 함수를 c# delegate로 할당 가능
* event
  * event callback 추가
  * event callback 삭제
* 64비트 정수
  * GC없이, 정밀도 손실없이 전달가능
  * lua 5.3에서 64비트 정수 기본지원
  * number와 혼합하여 연산 가능
  * java와 비슷한 방식으로 unsigned 64bit integer 지원
* lua table을 C#의 struct로 자동변환
  * obj.complexField = {a = 1, b = {c = 1}}，obj는 C# 오브젝트，complexField는 2층 중첩된 struct나 class.
* typeof
  * C#의 typeof operator에 대응됨, Type 객체 리턴
* lua측에서 직접 clone
* decimal
  * GC나 정밀도 손실없이 전달가능

## C#에서 Lua사용

* Lua 함수호출
  * delegate method로 Lua함수 호출
  * LuaFunction 객체로 Lua함수 호출
* Lua table접근
  * LuaTable의 Get/Set Template을 통한 접근, 접근시 gc가 필요하 않으며, 명확한 Key,Value의 타입을 지정할 수 있음.
  * CSharpCallLua표기된 interface의 접근
  * 값을 struct，class로 복사
 
## Lua 가상머신

* 가상머신 gc인자의 접근 및 설정

## Toolchain

* Lua Profiler
  * 함수의 총 사용시간, 평균시간, 호출 횟수등의 순서에 근거함
  * lua 함수명과 파일명 및 행수 표시
  * C#함수라면 C#함수라 표기됨
* 머신 디버깅 지원

