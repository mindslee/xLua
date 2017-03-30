![](Assets/XLua/Doc/xLua.png)

[![license](https://img.shields.io/badge/license-MIT-blue.png)](https://github.com/Tencent/xLua/blob/master/LICENSE.TXT)
[![release](https://img.shields.io/badge/release-v2.1.6-blue.png)](https://github.com/Tencent/xLua/releases)
[![PRs Welcome](https://img.shields.io/badge/PRs-welcome-blue.png)](https://github.com/Tencent/xLua/pulls)

## C#에 Lua 스크립트 지원

xLua는 Unity, .Net, Mono등의 C#환경에 Lua스크립트를 사용하도록 지원하며, 이 Lua코드는 C#과 상호도용이 가능합니다.

## xLua의 특장점

xLua는 기능, 성능, 사용성 모든 면에서 장점이 있습니다. 다음 몇가지 방면에서 특수성을 가집니다.

* C#코드(메소드, 오퍼레이터, 프로퍼티, 이벤트 등등)을 Lua코드로 대체하여 실행할 수 있습니다.
* 특별한 GC 최적화로 사용자정의 struct, enum등을 C# GC alloc 없이 Lua와 C#간에 전달할 수 있습니다.
* 에디터상에서 코드를 작성할 필요가 없어, 개발이 더 가벼워짐

상세한 내용 및 플랫폼지원 소개는 [이것](Assets/XLua/Doc/features.md)을 참고 바랍니다.

## 설치

압축을 풀어서 바로 사용가능. 맨 처음 사용시 샘플 패키지를 설치하시고 실행하여 효과를 보시기 바랍니다.

다른 디렉토리에 설치하기를 원한다면 [FAQ](Assets/XLua/Doc/faq.md) 관련 내용을 참고 바랍니다.

## lua5.3 vs luajit

xLua는 lua 5.3과 luajit의 두가지 버전이 있으며, 하나의 프로젝트는 한가지만을 선택할 수 있습니다. 이 두 버전의 C#코드는 동일하며, Plugins 부분만 다릅니다.

lua5.3는 64비트 정수, Apple bitcode, utf8지원 등의 더 많은 기능을 가집니다. c function내에서의 에러도 정확한 위치를 찾아줍니다. luajit에 비해 lua가 설치패키지에 주는 영향이 적습니다.

그러나 luajit는 장점은 성능입니다. 현재 luajit는 제작자가 luajit의 업데이트를 중단한 상태이며, 업데이트를 할 사람을 찾는 중이어서 이후의 개발이 불투명한 상태입니다.

프로젝트의 상황에 따라 판단하여 더 적합한것을 선택할 수 있습니다. 현재로는 lua5.3 버전의 사용이 좀더 많기 때문에, xLua프로젝트의 Plugins 디렉토리에는 기본으로 Lua 5.3버전은 포함하고 있습니다.

## Quick Start

3줄이면 완전한 코드가 됩니다.

xLua를 다운로드 받아 Unity 프로젝트의 Assets 디렉토리에 풀어넣고, MonoBehaviour를 Scene에 drag하여, Start에 다음의 코드를 삽입합니다.

```csharp
XLua.LuaEnv luaenv = new XLua.LuaEnv();
luaenv.DoString("CS.UnityEngine.Debug.Log('hello world')");
luaenv.Dispose();
```

1. DoString의 인자는 string이며 임의의 Lua코드를 입력할 수 있습니다. 이 예제에서는 Lua상에서 C#의 UnityEngine.Debug.Log를 사용하고 있습니다.

2.  하나의 LuaEnv는 하나의 Lua가상머신에 대응합니다. 프로젝트에 따라 다르겠지만 글로벌로 하나만 사용하기를 권장합니다.

C#에서 Lua를 사용하는것도 매우 간단합니다. 예를들어 Lua의 시스템 함수를 사용하는 추천방식은 다음과 같습니다.

* 선언

```csharp
[XLua.CSharpCallLua]
public delegate double LuaMax(double a, double b);
```

* 바인딩

```csharp
var max = luaenv.Global.GetInPath<LuaMax>("math.max");
```

* 사용

```csharp
Debug.Log("max:" + max(32, 12));
```

바인딩은 1회만 하는것을 추천합니다. 코드를 생성한 후에 max를 사용할 때에는 gc alloc을 생산하지 않습니다.

## 핫픽스

* 작은 수정. 기존 코드의 수정없이 사용이 가능합니다.
* 운영시 영향이 적음. 베이스 및 기존 코드를 패치할 필요가 없음
* 버그 발생시, Lua를 이용해 패치를 할 수 있으며, 이때에서야 Lua코드 로직으로 진입합니다.

[여기](Assets/XLua/Doc/hotfix.md)에 관련 내용이 있습니다.

## 더 많은 예제

* [01_Helloworld](Assets/XLua/Examples/01_Helloworld/): 입문 예제
* [02_U3DScripting](Assets/XLua/Examples/02_U3DScripting/): lua에서 어떻게 MonoBehaviour를 작성하는지 예제
* [03_UIEvent](Assets/XLua/Examples/03_UIEvent/): Lua에서 어떻게 UI로직을 작성하는지 예제
* [04_LuaObjectOrented](Assets/XLua/Examples/04_LuaObjectOrented/):  Lua 객체지향과 C#의 연동 예제
* [05_NoGc](Assets/XLua/Examples/05_NoGc/): GC를 피하는 예제
* [06_Coroutine](Assets/XLua/Examples/06_Coroutine/): Lua coroutine과 Unity coroutine의 상호연동
* [07_AsyncTest](Assets/XLua/Examples/07_AsyncTest/): Lua coroutine 동기화 예제
* [08_Hotfix](Assets/XLua/Examples/08_Hotfix/): 핫픽스 예제 (핫픽스를 사용하기 위해 다음의 [문서](Assets/XLua/Doc/hotfix.md)를 참조하시오)
* [09_GenericMethod](Assets/XLua/Examples/09_GenericMethod/): 제너릭함수 지원 예제
* [10_SignatureLoader](Assets/XLua/Examples/10_SignatureLoader/): 코드암호화 관련 예제. [문서](Assets/XLua/Doc/signature.md)참조
 
## 문서

* [XLua튜토리얼.docx](Assets/XLua/Doc/XLua튜토리얼.docx)：튜토리얼, [예제코드](Assets/XLua/Tutorial/).
* [XLua的配置.doc](Assets/XLua/Doc/XLua的配置.doc)：xLua 설정에 관하여
* [XLua增加删除第三方lua库.doc](Assets/XLua/Doc/XLua增加删除第三方lua库.doc)： 서드파티 lua확장라이브러리 추가에 관하여.
* [XLua API.doc](Assets/XLua/Doc/XLua_API.doc)：xLua API문서.
* [生成引擎二次开发指南](Assets/XLua/Doc/custom_generate.md)：코드생성 엔진의 2차개발에 관하여
* [热补丁操作指南](Assets/XLua/Doc/hotfix.md)：핫픽스 기능의 사용에 관해

## 기술지원 技术支持

QQ群：612705778 验证答案：有问题先找FAQ


