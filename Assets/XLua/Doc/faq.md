# FAQ

## xLua는 어떻게 사용하는가?

xLua 현재 zip으로 압축되어 배포된다. 프로젝트 디렉토리에 압축을 풀어 놓으면 된다.

## xLua는 다른 디렉토리에 설치해도 되는가?

그렇다. 그러나 CodeGeneration 디렉토리는 설정을 해줘야 한다. (기본적으로는 Assets\XLua\Gen), 구체적인 내용은 <XLua의설정.docx>의 GenPath설정 내용을 참고하시오.

## lua소스코드는 .txt 여야만 하는가?

어던 확장자여도 상관없다

만일 TextAsset으로 빌드를 하려면(예르르들어 Resources디렉토리), Unity가 .lua 확장자를 인식할수 없다. 이것을 Unity의 규칙이다.

만약 설치패키지에 포함하지 않는다면, 확장자 규칙은 없다. 예를들어 어떤 디렉토리에 다운로드 받은후, CustomLoader나 package.path에 설정을 통해 이 디렉토리에서 로드할 수 있다.

그런 어째서 xLua가 가진 lua코드들은 .txt확장자를 가지고 있는가? 왜냐하면 xLua자체는 하나의 라이브러리이고, Download기능을 가지고 있지 않으며, 런타임중에 다운로드 받을수도 없기떄문에 TextAsset을 통해 간단하게 처리하기 때문이다.

## Plugins소스코드는 어디에 있으면, 어떻게 사용하는가?

Plugins 소스코드는 xLua_Project_Root/build에 있다.

소스코드는 cmake로 컴파일되며, cmake설치후 make_xxxx_yyyy.zz을 실행한다. xxxx는 플랫폼(ios，android등), yyyy는 사용하는 가상머신(lua53과 luajit중에 선택), zz는 확장자(windows는 bat, 기타 sh).

windows는 Visual Studio 2015 에 의존한다.

android 컴파일은 linux에서 진행하며, NDK가 필요하다. 그리고 스크립트 내 ANDROID_NDK를 NDK설치 디렉토리로 변경해야한다.

ios와 osx은 mac에서 컴파일한다.

## "xlua.access, no field __Hitfix0_Update"의 에러는 어떻게 해결하는가?

[Hotfix 가이드](hotfix.md)를 잘 보고 진행하시오.

## "please install the Tools"

Tools가 Assets 디렉토리에 없음. 설치패키지 또는 master에서 찾을 수 있다.

## "This delegate/interface must add to CSharpCallLua : XXX" exception은 어떻게 해결하는가?

유니티에디터에서 xLua는 코드를 생성하지 않아도 다 동작한다. 이 메시지가 나왔다는것은 CSharpCallLua를 추가하지 않았거나, 추가한 후 코드를 재생성 하지 않은것이다.

해결방법으로는, 클래스 선언에 [CSharpCallLua]를 추가후, 코드를 클리어 후 다시 생성

## hotfix에서 어떻게 event를 발생시키는가?

우선 xlua.private_accessible를 통해 private멤버 방문을 허용한다.

객체의 "&이벤트명" 필드를 통해 delegate를 사용하다. 예를들어 self\['&MyEvent'\](). 여기서 MyEvent가 이벤트명이다.

## Unity Coroutine의 구현메소드를 어떻게 패치하는가?

[Hotfix가이드](hotfix.md)에 관련 장이 있음.

## NGUI(또는 UGUI/DOTween 등등)을 지원하는가?

지원한다, xLua는 주요한 특징은 C#으로 작성된 코드를 lua코드로 대체하는것이다. C#에서 사용할 수 있는 Plugin은 기본적으로 모두 사용가능하다.

## 디버깅이 필요할때, CustomLoader의 filepath인자는 어떻게 처리되는가?

lua에서 require 'a.b'사용시, CustomLoader가 사용되며, "a.b"스트링이 전달된다. 이때 이 스트링을 이해해야 하며(문서인지, 메모리인지, 네트워크주소인지 등등), lua 파일을 잘 로드하고, 두개의 값을 리턴한다. 하나는 디버거가 인식할 수 있는 path(예를 들어 a/b.lua, 이것은 ref type의 filepath인자로 반환된다)이고, 또다른 하나는 utf8형식 소스의 스트림(byte[])이다. 이것은 return값을 통해 반환된다.

## Generate Code란 무엇인가?

xLua에서 지원하는 Lua와 C#간의 상호 통신방식중에 하나이다. 이 방식은 양자간의 바인딩코드를 생성하여 구현되며, 성능이 좋고 추천하는 방식이다.

또다른 방식은 Reflection을 사용하는 것인데, 이 방법은 설치패키지에 영향이 더 적고, 성능요구도 크지않기 때문에, 설치패키지 크기에 민감한 경우에 사용할 수 있다.

## interface 변경후, 이전에 생성된 코드에서 오류가 발생하는데 어떻게 하는가?

생성된 코드를 삭제하라. ("Clear Generated Code" 메뉴를 실행하라. 만약 재실행후 이 메뉴를 찾지 못한다면, 수동으로 코드생성 디렉토리를 삭제할수 있다.) 컴파일이 완료된다 다시 생성하라.

## 언제 Generate Code를 해야 하는가?

개발중에는 Generate를 하지 않는것을 추천한다. 그래야 수많은 이유의 컴파일 오류를 피할 수 있으며, 생성된 코드의 컴파일도 기다려야 하기때문이다.

폰버전 빌드 전에 반드시 Generate Code를 수행해야하며, 자동화 하길 권장한다.

성능 프로파일이나 테스트를 하기전에 Generate Code를 반드시 수행해야 한다. 왜냐하면 한것과 안한것의 차이가 매우 크기 때문이다.

## CS namespace에 모든 C# API가 있는데 메모리를 많이 점유하는것은 아닌가?

lazyload를 하기때문에, 사용할 때만 생성된다. 예를 들어 UnityEngine.GameObject은 맨처음 CS.UnityEngine.GameObject 또는 처음 Example을 lua에 전달될때 해당 클래스의 method와 property가 로드된다.

## 어떤상황에서 LuaCallSharp를 쓰고 어떤 상황에서 CSharpCallLua를 사용하는가?

사용하는 쪽인지 사용되는쪽인지를. 예를들어 lua에서 C#의 GameObject.Find함수를 사용하거나 GameObject에 구현된 메소드, 속성등을 사용하려면, GameObject클래스에는 LuaCallSharp을 추가해야한다. 그리고 lua함수를 UI callback으로 사용할 경우 이것은 C#이 사용자이고, lua함수가 사용되어지는 것이다. 그러므로 callback으로 이 되는 delegate는 CSharpCallLua를 사용한다.

이것이 가끔을 헤깔릴 수가 있는데, 예를들어 List<int>.Find(Predicate<int> match)를 사용시, List<int>는 당연히 LuaCallSharp를 추가하고, Predicate<int>는 CSharpCallLua를 추가해야한다. 왜냐하면 match를 호출하는쪽은 C#이며, 호출되는쪽은 lua함수 이기 때문이다.

좀더 쉽게 생각하면 "This delegate/interface must add to CSharpCallLua : XXX” 메시지를 보면 XXX에 CSharpCallLua를 추가하면 된다.

## 값 타입 전달시 gc alloc이 발생하는가?

만약 delegate로 lua함수를 사용하거나 LuaTable, LuaFunction의 무 GC 인터페이스나, array, value 타입을 사용할 경우 GC가 발생하지 않는다. 

1. 모든 기본 값 타입 (모든 정수형, 실수형, decimal)

2. 모든 열거형

3. 필드에 값 타입으로 이루어진 struct, 또다른 값 타입으로 된 struct를 중첩하여 가진 struct

이중 2, 3은 해당 타입을 GCOptimize에 추가해야한다.

## Reflection을 iOS에서 사용가능한가?

ios에서는 두가지 제한이 있다. 하나는 jit가 없다는것이고, 두번째는 code stripping이다.

C#에서 delegate나 interface를 통해 Lua를 사용할때, Code Generation을 해놓지 않았으면 Reflection emit를 사용한다. 이것은 lazy jit이기 때문에 유니티 에디터에서만 사용가능하다.

Lua에서 C#을 사용시에는, code stripping에 영향을 받을 수 있다. 이 경우 ReflectionUse를 표기할수 있다.(LuaCallSharp를 표기하지 않는다)，"Generate Code"시 해당 클래스는 바인딩 코드를 생서하지 않을것이며, link.xml에 해당 클래스의 code stripping을 안하도록 설정하게 될 것이다.

간단히 말해, CSharpCallLua는 꼭 필요한것에만 사용하고(이 유형은 일반적으로 많지 않다), LuaCallSharp 생성은 모두 Reflection을 사용하도록 할 수 있다.

## Generic Method를 사용할수 있는가?

직접지원은 아니지만 사용할 수 있다. 만약 static method라면 해당 제너릭메소드의 래퍼코드를 직접 작성할 수 있다.

만약 member method라면, xLua는 확장 메소드를 지원하므로, 확장 메소드로 제너릭 메소드를 구현을 추가할 수 있다. 확장 메소드를 사용하는 방법은 일반 멤버 메소드를 사용하는것과 동일하다.

```csharp
// C#
public static Button GetButton(this GameObject go)
{
    return go.GetComponent<Button>();
}
```

```lua
-- lua
local go = CS.UnityEngine.GameObject.Find("button")
go:GetButton().onClick:AddListener(function()
    print('onClick')
end)
```

## Lua에서 C#의 오버로딩 함수를 지원하는가?

지원한다. 그러나 C#에서 지원하는것 처럼 완벽하지는 않다, 예를들어 오버로드 함수 void Foo(int a)와 void Foo(short a)가 있을때, int와 short는 모두 lua의 number에 대응되기 때문에 인자에 따라 어떤 함수를 호출하는지 판단할 수가 없다. 이것은 확장메소드를 통해 각각의 별명을 지어 해결하면 된다.

## 유니티 에디터상에서는 정상으로 동작하는데, 빌드를 하려하면, "어떤 메소드, 프로퍼티, 필드정의가 없다" 라는 메시지가 나온다 어떻게 하는가?

종종 어떤 메소드, 프로퍼티, 필드가 매크로 조건에 따라 동작하기도 한다(UNITY_EDITOR에서만 유효한). 이것은 이런 메소드, 프로퍼티, 필드를 블랙리스트에 등록하여 해결한다. 추가후 컴파일이 완료된 후에 코드생성을 다시 수행한다.

## this[string field] 또는 this[object field] operator overload는 lua에서 왜 사용이 불가한가? (예 Dictionary\<string, xxx\>, Dictionary\<object, xxx\>는 Lua에서 dic['abc']또는 dic.abc의 방법으로 사용할 수 없다)

2.1.5~2.1.6 버전에서 이 기능이 제거되었다. 왜냐하면 첫째 이 기능은 베이스클래스에 정의된 메소드, 프로퍼티 필드에 접근하지 못하는 상황을 발생시킬 수 있다. (예를 들어 Animation에서 GetComponent 메소드에 접근이 안됨） 둘째, key를 객체로 하게되면 메소드, 프로퍼티, 필드의 데이터에 접근할 수가 없다. 예를들어 Dictionary에서 dic['TryGetValue']가 함수를 리턴하면, Dictionary의 TryGetValue메소드를 접근하게 된다.

그러므로 해당 operator와 동일한 기능의 메소드를 사용하는것을 추천한다. 예를들어 List의 Get, Dictionary의 TryGetValue. 만약 그런 메소드가 존재하지 않는다면, C#에 Extension method를 추가하여 사용하면 된다.

## C#에서 null인 Unity객체가 lua에서는 어째서 nil이 아닌가? 예를들어 이미 Destroy한 GameObject.

사실 그 C#객체는 null이 아니고, UnityEngine.Object가 == operator를 오버로드하여 그렇게 나오는것이다. Destory했거나 아직 초기화 하지 않은 객체에서, obj == null는 true를 리턴한다. 하지만 이 C# 오브젝트는 null이 아니다. System.Object.ReferenceEquals(null, obj)을 통해 확인할 수 있다.

이런 상황에 대비하여 UnityEngine.Object에 다음의 Extension을 사용할 수 있다.

~~~csharp
[LuaCallCSharp]
[ReflectionUse]
public static class UnityEngineObjectExtention
{
    public static bool IsNull(this UnityEngine.Object o) // 或者名字叫IsDestroyed等等
    {
        return o == null;
    }
}
~~~

그러면 Lua에서 모든 UnityEngine.Object에 IsNull 판단을 할 수 있다.

~~~lua
print(go:GetComponent('Animator'):IsNull())
~~~

## 제너릭 표현은 어떻게 구성하는가?

제너릭표현의 구성은 보통의 타입과 동일하다. 모두 CS.namespace.typename()이다. 조금 특히한것은 typename의 표현일 것이다. 제너릭의 typename의 표현은 비정규 심볼을 포함한다. 마지막의 일부분은 ["typename"]로 바뀐다. List<string>의 예를보면

~~~lua
local lst = CS.System.Collections.Generic["List`1[System.String]"]()
~~~

만약 어떤 제너릭의 typename을 잘 모른다면, C#에서 typeof().ToString()을 통해 확인하면된다.

