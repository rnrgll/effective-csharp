# 13\. 정적 클래스 멤버를 올바르게 초기화하라

정적 멤버 변수를 포함하는 객체의 경우 인스턴스 생성 전 정적 멤버 변수를 초기화해야 한다.

정적 멤버를 초기화하는 방법은 두 가지가 있다.

*   정적 멤버 초기화 구문
*   정적 생성자

## 13.1 정적 멤버 초기화 구문

```csharp
public class MySingleton
{
	private static readonly MySingleton theOneAndOnly = new MySingleton(); // 정적 멤버 초기화 구문

	public static MySingleton TheOnly { get{ return theOneAndOnly;} }
	private MySingleton() {}
}
```

*   해당 타입(클래스)이 처음 사용되기 직전, CLR의 타입 초기화 과정에서 실행된다.
*   기본적으로 한 번만 실행되고, 이후에는 값이 유지된다.

## 13.2 정적 생성자

타입 내에 정의된 모든 메서드, 변수, 속성에 **최초로 접근하기 전에 자동으로 호출되는 메서드.**

```csharp
public class MySingleton
{
	private static readonly MySingleton theOneAndOnly;

	static MySingleton() // 정적 생성자
	{ 
		theOneAndOnly = new MySingleton(); 
	}

	public static MySingleton TheOnly { get{ return theOneAndOnly;} }	
	private MySingleton() {}
}
```

### 13.2.1 특징

*   접근 제한자(`private`, `public`)을 붙일 수 없다.
*   매개변수를 가질 수 없다.
*   직접 호출할 수 없다.
*   CLR이 타입 초기화(type initializtion) 과정에서 알아서 호출한다.
*   딱 한 번만 실행된다.
*   하나 타입은 하나의 정적 생성자만 가질 수 있다.

### 13.2.2 초기화 순서

*   정적 멤버 초기화 구문은 정적 생성자 시작 부분에 포함되므로 먼저 실행된다.
*   부모 클래스의 정적 생성자가 자식 클래스의 정적 생성자보다 먼저 호출된다.

## 13.3 정적 초기화 시 주의사항

### 13.3.1 인스턴스 생명 주기에 의존하면 안된다.

정적 변수 초기화는 반드시 **정적 멤버 초기화 구문 또는 정적 생성자를 통해 수행해야 한다.** 인스턴스 생성자에서 초기화 하면 안된다!

```csharp
public class Example
{
    private static int value;

    public Example()
    {
        value = 10; // X 잘못된 초기화 방식
    }
}

```

*   인스턴스를 생성해야만 초기화가 수행된다.
*   정적 멤버는 인스턴스 없이도 접근 가능하기 때문에, **사용 시점에 따라 초기화되지 않은 상태가 발생할 수 있다.**

### 13.3.2 예외 발생 가능성을 최소화해야 한다.

정적 생성자는 CLR에 의해 호출된다. CLR은 예외가 발생하면 `TypeInializationException`을 던지고 응용프로그램을 종료한다.

응용프로그램이 종료되지 않더라도,

*   타입 초기화가 실패한 것으로 간주된다.
*   해당 타입은 **재초기화를 시도하지 않는다.**
*   애플리케이션 도메인이 언로드되지 않는 한 그 타입을 다시 사용할 수 없다.

따라서 정적 생성자 및 정적 초기화 코드에서는 **예외가 발생할 가능성을 최소화**해야 한다.

## 13.4 Lazy\<T>

정적 필드 초기화는 CLR의 타입 초기화(type initializtion) 과정에서 실행된다. 이때 수행되는 **작업이 복잡하거나 많은 자원을 소비하는 경우 성능 문제**가 발생할 수 있다.

*   처음 쓰는 순간에 프레임 드랍
*   로딩 지연
*   앱 시작 시 튐 현상

이런 경우에는 Lazy<T>를 사용해 실제 사용 시점까지 초기화를 지연하는 것이 좋다.

<div class="textBoxWoTitle"><div class="textBoxContent"><p>❗클래스의 초기화를 미루는 것이 아니라, <strong>특정 정적 멤버의 실제 리소스 생성을 미루는 방식이다.</strong> 초기화할 때 가벼운 객체만 생성하고 실제로 그걸 사용할 때 실제 자원을 생성한다.</p></div></div>

### 13.4.1 Lazy<T> 내부

단순화한 구조

```csharp
class Lazy<T>
{
    private Func<T> _factory;
    private T _value;
    private bool _initialized;

    public T Value
    {
        get
        {
            if (!_initialized)
            {
                _value = _factory();
                _initialized = true;
            }
            return _value;
        }
    }
}

```

### 13.4.2 Lazy<T> 초기화 과정

```csharp
class ItemDB
{
    private static readonly Lazy<Dictionary<int, string>> _items
        = new Lazy<Dictionary<int, string>>(() =>
        {
            // 무거운 로딩 작업
            var dict = new Dictionary<int, string>();
            dict[1] = "Potion";
            return dict;
        });

		static ItemDB()
		{
		    Console.WriteLine("ItemDB static ctor");
		}
		
    public static string GetName(int id) => _items.Value[id];
}
```

*   타입 초기화 시점에서는 Lazy 객체만 생성되고, `_factory` 만 세팅되고 `_value` 는 설정되지 않는다.
    *   즉, 이 시점에서 `Dictionary<int, string>` 는 아직 생성되지 않았다.

```csharp
var name = ItemDB.GetName(1);
```

*   이후 `_items.Value` 에 접근하여 \_**items에 실제로 접근/사용하는 순간 Dictionary 생성/로딩이 된다.**

### 13.4.3 Lazy\<T> vs 프로퍼티 get 지연 초기화

정적 리소스를 실제 사용하는 시점까지 초기화를 미루기 위해 가장 흔히 사용되는 두 가지 방식은 다음과 같다.

*   프로퍼티 `get`을 이용한 지연 초기화
*   `Lazy<T>`를 이용한 지연 초기화

#### 프로퍼티 get 기반 지연 초기화

##### 동작 방식

```csharp
static Dictionary<int, string> _items;

static Dictionary<int, string> Items
{
    get
    {
        if (_items == null)
        {
            _items = LoadItems();
        }
        return _items;
    }
}

```

*   `_items`는 초기에는 `null` 상태
*   `Items` 프로퍼티가 처음 호출될 때
    *   `LoadItems()`를 실행하여 실제 객체를 생성
    *   이후에는 생성된 객체를 그대로 재사용

##### 문제점 (멀티스레드 환경)

```csharp
Thread A: _items == null → LoadItems()
Thread B: _items == null → LoadItems()

```

*   중복 초기화 문제 : 동일한 리소스가 두 번 생성될 수 있다.
*   경쟁 조건 : 여러 스레드가 동시에 `_items` 상태를 검사하고 변경할 수 있다.
*   반쯤 초기화된 상태가 노출되는 문제 : 한 스레드가 초기화 중인 객체를 다른 스레드가 먼저 접근할 수 있다.

→ `lock` 을 추가하거나 복잡한 동기화 코드를 직접 작성해야 한다.

#### Lazy\<T>

##### 특징

*   스레드 안전하다. 여러 스레드가 동시에 접근해도 <span class="fontColorRed">초기화 로직은 단 한번만 실행된다.</span>
*   내부적으로 <span class="fontColorRed"> lock / 상태 플래그 등을 사용해 안전성을 보장</span>한다.

→ 멀티스레드 환경에서 **추가 코드 없이 안전한 지연 초기화**를 제공하는 것이 가장 큰 장점이다.

#### Unity 환경에서 실질적 차이

Unity의 대부분의 API는 **메인 스레드에서만 호출 가능하도록 설계되어 있다.** 이로 인해 정적 필드 접근 또한 일반적으로 메인 스레드에서만 이루어진다.

이러한 환경에서는,

*   동시에 여러 스레드가 정적 필드에 접근할 가능성이 낮고
*   경쟁 조건(Race Condition)이 발생할 여지가 거의 없다

따라서 **프로퍼티 `get`을 이용한 지연 초기화 방식도 실질적으로 안전하게 동작**하며, `Lazy<T>` 가 제공하는 스레드 안정성의 이점이 크게 체감되지 않는다.

다만, 다음과 같은 환경에서는 `Lazy<T>` 방식이 더 안정적이다.


*  백그라운드 스레드를 사용하는 경우
    *   데이터 로딩
    *   네트워크 처리
    *   비동기 작업 (`Task`, Job System 등)
*   메인 스레드 외부에서 정적 리소스에 접근할 가능성이 있는 구조
