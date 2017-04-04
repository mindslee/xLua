## 사용방식

1. 이 기능은 기본적으로 꺼져있으며, HOTFIX_ENABLE 매크로를 정의해야함 (Unity3D에서 File->Build Setting->Scripting Define Symbols에 추가). 폰빌드를 뽑을때 잊으면 안됨!!

2. 현재 버전은 지연 코드생성이다. HotFix는 XLua/Generate Code를 수행해야만 정상적으로 동작한다.(추천 개발방식은 평소에는 HOTFIX_ENABLE을 켜지 않고 - 이때는 바인딩 코드생성을 할필요없다 - 폰 빌드나 유니티에서 HOTFIX를 할때만 HOTFIX_ENABLE을 켜서 사용한다）

3. 유니티에디터에서 "XLua/Hotfix Inject In Editor"을 실행해야한다. 만약 "hotfix inject finish!"이나 "had injected!"가 출력되면, injection이 성공한것이다. 패치 테스트를 할수 있다.(폰빌드는 이 단계가 필요없음) 만약 자동빌드 시스템을 사용한다면, 코드내 사용된 API설정 매크로는 무효가 될수있다. 에디터상에서 대응되는 플랫폼에도 설정을 해야한다.

## Embedded Mode

기본적으로는 Tool을 통해 코드 Injection을 수행하며, 에디터상에 임베디드 하는 방법을 사용할 수도 있다. INJECT_WITHOUT_TOOL 매크로를 정의하면 된다.

INJECT_WITHOUT_TOOL 매크로 정의후, Hotfix기능은 Cecil에 의존한다. HOTFIX_ENABLE 매크로 추가 후, Cecil을 찾을수 없을수도 있다. 이때는 Unity 설치 디렉토리에서 Mono.Cecil.dll, Mono.Cecil.Pdb.dll, Mono.Cecil.Mdb.dll을 찾아 프로젝트 목록에 복사해놓은면 된다.

주의 : 만약 Unity 설치 디렉토리에 Mono.Cecil.Pdb.dll, Mono.Cecil.Mdb.dll가 없다면(일부 구 버전에 없을수 있다.), 그럼 그냥 Mono.Cecil.dll만을 카피한다. 만약 다른 버전의 Unity에서 카피한다면, Unity 에디터가 불안정 할 수 있다. 이때는 HOTFIX_SYMBOLS_DISABLE를 정의해야 C#코드가 디버그가 안되거나 파일 라인수가 잘못되거나 하는 문제를 막을수 있다.(그러니 Unity버전을 올려라)

참고 명령 (Unity 버전에 따라 약간 다를 수 있다.)

```shell
OSX Command: cp /Applications/Unity/Unity.app/Contents/Managed/Mono.Cecil.* Project/Assets/XLua/Src/Editor/
Win Command: copy UnityPath\Editor\Data\Managed\Mono.Cecil.* Project\Assets\XLua\Src\Editor\
```

## 제한

static constructor 미지원

자식 클래스의 override된 함수에서 부모 클래스의 구현을 사용을 지원하지 않음

현재는 Assets 아래의 코드만을 지원하며, 엔진이나 C# 시스템 라이브러리의 Hotfix는 지원하지 않음.

## API
xlua.hotfix(class, [method_name], fix)

* description : lua 패치의 주입(injection)
* class       : C# 클래스, 두가지 표현방법 CS.Namespace.TypeName 또는 스트링 "Namespace.TypeName". String형식과 C#의 Type.GetType의 값과 일치해야함. 만약 Nested Type으로 Public클래스가 아니라면, "Namespace.TypeName+NestedTypeName"의 스트링 형식으로 할 수 밖에 없다.
* method_name : 메소드 네임. 선택가능
* fix         : method_name이 있다면 fix는 하나의 function, 그렇지 않으면 function들의 table이다. table 각 필드의 key는 method_name, value는 function의 method이다.

xlua.private_accessible(class)

* description : class의 private field, property, method를 사용가능하게 함.
* class       : xlua.hotfix의 class인자와 동일

## Hotfix

다른 설정들 처럼 두가지 방식이 있다.

1. 그냥 Class의 앞에 Hotfix표식을 함.

2. static클래스의 static 필드 및 프로퍼티의 앞에 설정함. property는 구현이 비교적 복작한 설정이다. 속성은 Namespace에 따른 화이트리스트를 작성하는 등의보다 복잡한 구성을 달성하기 위해 사용될 수 있다.

~~~csharp
public static class HotfixCfg
{
    [Hotfix]
    public static List<Type> by_field = new List<Type>()
    {
        typeof(HotFixSubClass),
        typeof(GenericClass<>),
    };

    [Hotfix]
    public static List<Type> by_property
    {
        get
        {
            return (from type in Assembly.Load("Assembly-CSharp").GetTypes()
                    where type.Namespace == "XXXX"
                    select type).ToList();
        }
    }
}
~~~

## 사용제안

* 모든 변화가능성이 있는 모든 Class에 Hotfix 표식을 하라.
* Reflection을 사용하게될 모든 함수인수, 필드, 속성, 이벤트에 사용하는 delegate에 CSharpCallLua 표식을 한다.
* 상업용코드, 엔진API, 시스템API는 Lua패치시 고성능이 필요하므로 LuaCallCSharp을 붙인다.
* 엔진API, 시스테API는 CodeStripping이 될수 있다. (C#는 사용하지 않는 모든 코드를 Stripping한다.), 만약 새로 추가한 C#코드외에 API를 사용할 필요가 있다면, 이 API를 사용하는 클래스에 LuaCallCSharp을 사용할 수 있다. 아니면 ReflectionUse를 사용하다.

## Stateless와 Stateful

Hotfix표식을 할 때, 기본적으로는 Stateless모드이다. 그리고 Stateful모드를 선택할 수 있다. 차이를 먼저 설명하고, 사용예제를 보기로 하자.

Stateless모드는 멤버함수가 Lua에 의해 복원될때, C# 객체값이 Lua함수의 첫번째 인자로 전달되는 모드이다.

Stateful모드에서는 Lua constructor함수에서 table을 반환한 후, 멤버함수는 이 테이블을 통해 사용한다.

Stateless 모드는 State가 없는 클래스에 적합하다. State가 있다면 Reflection을 통해 private 멤버에 접근해야하며, 새 상태(field)를 추가할 수도 없다. Stateless 모드는 장점은 운용 중 임의이 시점에 교체를 할수 있다는것이다.

Stateful의 대가는 LuaTable클래스의 필드를 추가할수 있다는데 있다(중간층에서의 추가이며, 원본코드를 고칠 필요가 없다). 단 이 모드는 확장성이 더 좋다, 만약 lua 상태가 필요없다면, constructor함수에서 nil을 리턴할 수 있다. 또한 리플렉션으로 private 변수에 접근하는것보다 성능면에서 좋으며, 임의의 상태정보를 추가할 수도 있다. 단점은 멤버함수를 사용하기 전에 이미 new된 객체사용하면 nil이 리턴될수 있어 반드시 재시작이 필요하므로, 시작시점에 교체를 해야한다.

## 패치만들기

xlua는 lua함수로 C#의 constructor, function, property, event를 대체할 수 있다. lua 구현은 모두 함수이다. 예를들어 property는 getter함수하나, setter함수 하나이고, event는 add함수 하나와 remove함수 하나에 대응된다.

* 함수

method_name에 함수명이며, override를 지원하고, override된 함수들은 모두 하나의 lua함수를 통해 전달된다.

예시:

```csharp

// fix해야할 C# 클래스
[Hotfix]
public class HotfixCalc
{
    public int Add(int a, int b)
    {
        return a - b;
    }

    public Vector3 Add(Vector3 a, Vector3 b)
    {
        return a - b;
    }
｝

```

```lua

xlua.hotfix(CS.HotfixCalc, 'Add', function(self, a, b)
    return a + b
end)

```

static function과 member function의 차이는 member function은 self인자가 추가된다는것이다. 이것은 self가 Stateless 모드에서 C# 객체 자체(C#의 this)이고, Stateful방식에서는 lua constructor함수에서의 반환값으로 전달한다(table이거나 nil).

보통의 인자는 lua인자 하나에 대응되고, ref인자는 하나의 인자와 하나의 리턴값에 대응되고, out인자는 하나의 리턴값에 대응된다.

제너릭함수의 패치방법은 일반 함수와 동일하다.

* constructor

constructor 함수의 method_name은 ".ctor"이다.

만약 Stateful모드라면, 객체의 상태를 table로 리턴할 수 있다.

일반 함수와 다른 점은 constructor함수의 hotfix는 교체가 아니며, 이미 있는 로직의 실행 후에 lua를 호출하는것이다.

* property

Property명이 "AProp"라면 대응되는 getter 메소드의 method_name은 get_AProp이고 setter의 method_name는 set_AProp이다.

* [] operator

set는 set_Item에 대응되며, get은 get_Item에 대응된다. 첫번째 인자는 self이며, 그다음은 key, value이다. get의 경우는 key만 사용되고, value가 return된다.

* 기타 operator

C#의 operator는 모두 대응되는 operator함수명이 있다. 예를들어 +의 operator함수명은 op_Addition이다. (기타 operator의 함수명은 관련 자료를 참고바람), 이 함수를 override하는것은 C#의 +오퍼레이터를 override하게 된다.

* Event

예를들어 Event이름이 "AEvent"라면, += operator는 add_AEvent, -= operator는 remove_AEvent에 대응된다. 이 두개의 함수의 첫번째 인자는 self이며, 두번째 인자는 operator 뒤의 delegate이다.

xlua.private_accessible통해 이벤트에서 대응되는 private delegate의 직접 접근하려면, 객체의 "&이벤트명" 필드값을 통해 이벤트를 발생시킬수 있다.예를들어 self\['&MyEvent'\](), 여기서 MyEvent는 이벤트명이다.

* Destructor

method_name은 "Finalize"이며 self인자가 전달된다.

보통의 함수와 다른점은 destructor의 hotfix는 교체가 아니며, lua함수가 사용된 후에 원래의 로직이 동작하게된다.

* 제너릭 클래스

일반 클래스와 동일하지만 다른 점은 각 제너릭클래스는 사용될 때 하나의 독립된 클래스라는것이다. 只能针对实例化后的类型分别打补丁。比如：

```csharp
public class GenericClass<T>
{
｝
```

GenericClass\<double\>, GenericClass\<int\>이런 유형의 클래스만 가능하며, 또한 GenericClass에 hotfix를 하는것이 아니다.

또 하나 알아두어야 할 것은 제너릭 클래스의 네이밍이다. 예를들어 GenericClass\<double\>의 네이밍은 GenericClass`1[System.Double]이다. 구체적인 내용은 [MSDN](https://msdn.microsoft.com/en-us/library/w3f99sx1.aspx)를 참고 바람.

GenericClass<double>의 Hotfix 예제:

```csharp
luaenv.DoString(@"
    xlua.hotfix(CS['GenericClass`1[System.Double]'], {
        ['.ctor'] = function(obj, a)
            print('GenericClass<double>', obj, a)
        end;
        Func1 = function(obj)
            print('GenericClass<double>.Func1', obj)
        end;
        Func2 = function(obj)
            print('GenericClass<double>.Func2', obj)
            return 1314
        end
    })
");
```

* Unity coroutine

util.cs_generator를 통해 하나의 함수로 IEnumerator를 흉내낼수 있다. Lua에서 사용한 coroutine.yield는 C#의 yield return과 유사하다. 예를들어 아래의 C# Code와 대응되는 Hotfix코드는 동일한 효과이다.

~~~csharp
[XLua.Hotfix]
public class HotFixSubClass : MonoBehaviour {
    IEnumerator Start()
    {
        while (true)
        {
            yield return new WaitForSeconds(3);
            Debug.Log("Wait for 3 seconds");
        }
    }
}
~~~

~~~csharp
luaenv.DoString(@"
    local util = require 'xlua.util'
	xlua.hotfix(CS.HotFixSubClass,{
		Start = function(self)
			return util.cs_generator(function()
			    while true do
				    coroutine.yield(CS.UnityEngine.WaitForSeconds(3))
                    print('Wait for 3 seconds')
                end				
			end
		end;
	})
");
~~~

* 전체 클래스

만약 전체 클래스를 교체하려면, 매번 xlua.hotfix를 호출할 필요없이, 한번에 전체를 넘긴다. table로 넘기며, method_name = function 형식이다.

```lua

xlua.hotfix(CS.StatefullTest, {
    ['.ctor'] = function(csobj)
        return {evt = {}, start = 0}
    end;
    set_AProp = function(self, v)
        print('set_AProp', v)
        self.AProp = v
    end;
    get_AProp = function(self)
        return self.AProp
    end;
    get_Item = function(self, k)
        print('get_Item', k)
        return 1024
    end;
    set_Item = function(self, k, v)
        print('set_Item', k, v)
    end;
    add_AEvent = function(self, cb)
        print('add_AEvent', cb)
        table.insert(self.evt, cb)
    end;
    remove_AEvent = function(self, cb)
       print('remove_AEvent', cb)
       for i, v in ipairs(self.evt) do
           if v == cb then
               table.remove(self.evt, i)
               break
           end
       end
    end;
    Start = function(self)
        print('Start')
        for _, cb in ipairs(self.evt) do
            cb(self.start, 2)
        end
        self.start = self.start + 1
    end;
    StaticFunc = function(a, b, c)
       print(a, b, c)
    end;
    Finalize = function(self)
       print('Finalize', self)
    end
})

```

