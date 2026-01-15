# 14\. 초기화 코드가 중복되는 것을 최소화해라

여러 생성자를 정의할 때 초기화 코드가 반복되기 쉽다. C#에서는 생성자 호출 체인과 언어 기능을 활용하여 **중복 초기화 코드를 최소화하는 것이 바람직하다.**

## 14.1 공통 유틸리티 메서드 (권장 X)

C++에서는 공통 유틸리티 메서드(private 헬퍼 메서드)를 사용해 여러 생성자에서 공통 초기화 로직을 분리하는 경우가 많다.

하지만 **C#에서는 이 방식이 권장되지 않는다.**

### 예제

```csharp
public class MyClass
{
	private List<ImportantData> coll; // 데이터 컬렉션
	private string name; 
	public MyClass()
	{
			commonConstructor(0, "");
	}
	public MyClass(int initialCount)
	{
			commonConstructor(initialCount, "");
	}
	public MyClass(int initialCount, string Name) 
	{
			commonConstructor(initialCount, Name);
	 }
	 
	 private void commonConstructor(int count, string name)
	 {
			 coll = (initialCount > 0)?
						new List<ImportantData>(initialCount) :
						new List<ImportantData>();
				this.name = name;
	 }
}
```

공통 부분을 분리해서 코드 중복을 줄인 것처럼 보일 수 있지만 실제로는 다음과 같은 문제가 있다.

### 문제 1. 생성자 수준의 중복은 제거되지 않는다

C# 컴파일러는 **각 생성자마다** 다음 코드를 자동으로 삽입한다.

*   인스턴스 필드에 대한 기본 초기화 코드
*   베이스 클래스 생성자 호출
*   생성자 본문 코드

그 결과 아래와 같은 코드와 같다고 볼 수 있다. 코드의 중복을 제거하지 못한다.

```csharp
public class MyClass
{
	private List<ImportantData> coll; // 데이터 컬렉션
	private string name; 
	public MyClass()
	{
			.... // 인스턴스 필드 초기화 코드
			... // 베이스 클래스의 생성자 호출 코드 
			commonConstructor(0, ""); // 공용 유틸리티 함수
	}
	public MyClass(int initialCount)
	{
			.... // 인스턴스 필드 초기화 코드
			... // 베이스 클래스의 생성자 호출 코드 
			commonConstructor(initialCount, "");
	}
	public MyClass(int initialCount, string Name) 
	{
			.... // 인스턴스 필드 초기화 코드
			... // 베이스 클래스의 생성자 호출 코드 
			commonConstructor(initialCount, Name);
	 }
	 
	 private void commonConstructor(int count, string name)
	 {
			 coll = (initialCount > 0)?
						new List<ImportantData>(initialCount) :
						new List<ImportantData>();
				this.name = name;
	 }
}
```

### 문제 2 : readonly 멤버 필드 초기화 불가능

`readonly` 필드는 **멤버 초기화 구문 또는 생성자 내부에서만** 초기화할 수 있다.

```csharp
public class MyClass
{
	private List<ImportantData> coll; // 데이터 컬렉션
	private readonly string name; 
	public MyClass()
	{
			commonConstructor(0, "");
	}
	public MyClass(int initialCount)
	{
			commonConstructor(initialCount, "");
	}
	public MyClass(int initialCount, string Name) 
	{
			commonConstructor(initialCount, Name);
	 }
	 
	 private void commonConstructor(int count, string name)
	 {
			 coll = (initialCount > 0)?
						new List<ImportantData>(initialCount) :
						new List<ImportantData>();
				this.name = name; // X 컴파일 오류 발생
	 }
}
```

→ `CommonConstructor`는 생성자가 아니므로 생성자에서 호출되었더라도 `readonly` 필드를 초기화할 수 없다.

## 14.2 생성자 호출 체인 (공용 생성자 방식)

C#에서는 **생성자에서 다른 생성자를 호출하는 방식**을 제공한다.

이 방식을 사용하면 초기화 코드를 한 곳에 모을 수 있다.

```csharp
public class MyClass
{
	private List<ImportantData> coll; // 데이터 컬렉션
	private string name; 
	public MyClass() :  this(0, ""){}
	public MyClass(int initialCount) : this(initialCount, string.Empty) {}
	public MyClass(int initialCount, string name) 
	{
			coll = (initialCount > 0)?
						new List<ImportantData>(initialCount) :
						new List<ImportantData>();
			this.name = name;
	 }
}
```

실제로 아래와 같다.

```csharp
public class MyClass
{
	private List<ImportantData> coll; // 데이터 컬렉션
	private string name; 
	public MyClass()
	{
			// 변수 초기화 코드 X, 베이스 클래스 생성자 호출 코드 X
			this(0, "");  // (설명을 위한 코드, 유효한 코드 X)
	}
	public MyClass(int initialCount)
	{
			// 변수 초기화 코드 X, 베이스 클래스 생성자 호출 코드 X
			this(initialCount, "");  // (설명을 위한 코드, 유효한 코드 X)
	}
	public MyClass(int initialCount, string name) 
	{
	
			.... // 인스턴스 초기화 코드
			... // 베이스 클래스의 생성자 호출하는 코드 
			coll = (initialCount > 0)?
						new List<ImportantData>(initialCount) :
						new List<ImportantData>();
			this.name = name;
	 }
}
```

→ 실제 초기화 코드는 하나의 생성자에만 존재하며, 베이스 클래스 생성자 호출과 필드 초기화가 한 번만 수행된다. 또한, `readonly` 필드 초기화가 가능하다.

## 14.3 기본 매개변수(Default Parameter) 활용

C# 4.0에 추가된 기본 매개변수 기능을 활용하면 중복 코드를 더 줄일 수 있다.

위의 코드는 아래와 같이 기본 값을 가진 매개변수를 취하는 생성자 하나로 대체할 수 있다.

```csharp
public class MyClass
{
	private List<ImportantData> coll; // 데이터 컬렉션
	private string name; 
	public MyClass() :  this(0, ""){}
	public MyCkass(int initialCount = 0, string name = "") 
	{
			coll = (initialCount > 0)?
						new List<ImportantData>(initialCount) :
						new List<ImportantData>();
			this.name = name;
	 }
}
```

> #### **💡기본 매개변수 값이 <code>""</code> 인 이유**
> 
> 기본 매개변수는 <strong>컴파일 타임 상수만 사용 가능</strong>하다.
> 
> <code class="language-csharp">void Foo(string s =""){}</code>
>
> 컴파일러는 호출부를 다음과 같이 치환한다.
> 
> <code>Foo(); → Foo("");</code>
> 
> ⇒ 기본값은 호출 시점에 평가되는 것이 아니라 <strong>호출 코드에 그대로 박힌다</strong>
>
> <code>string.Empty</code> 는 <strong>런타임에 평가되는 정적 속성</strong>(string 클래스에 정의된 정적 속성)이므로 기본 매개변수로 사용할 수 없다.



### 기본 매개변수 사용 시 주의할 점

1.  **`new()` 제약 조건과의 충돌**
    
    `new()` 제약 조건을 명시한 제네릭 클래스와 함께 사용해야 할 경우
    
    ```csharp
    class Factory<T> where T : new()
    {
        public T Create()
        {
            return new T(); // 진짜 매개변수 없는 생성자 호출
        }
    }
    
    public class MyClass
    {
    		// 기본 생성자 없음..!!
    		
    		public MyClass(int initialCount = 0, string name = "")
    		{
    			 // ....
    		}
    }
    
    ```
    
    *   <span class="fontColorRed">기본 매개변수 생성자는 <strong>기본 생성자가 아니다</strong></span>
    *   `new()` 제약 조건을 만족하지 못하기 때문에 이러한 경우에는 명시적인 기본 생성자가 필요하다.
2.  **공개 인터페이스 결합도 증가**
    
    기본 매개변수의 값과 이름은 공개 API의 일부이기 때문에 외부 코드와의 결합도가 높아질 수 있다.
    
    → 매개변수 이름 변경 시 이를 사용하는 모든 호출 코드를 재컴파일해야 한다.
    

## 14.4 C#의 객체 초기화 과정

**특정 타입으로 첫 번째 인스턴스를 생성할 떄 수행되는 과정**

1.  정적 변수의 저장 공간을 0으로 초기화
2.  정적 변수에 대한 초기화 구문 수행
3.  부모 클래스의 정적 생성자 수행
4.  정적 생성자 수행
5.  인스턴스 변수의 저장 공간을 0으로 초기화
6.  인스턴스 변수에 대한 조기화 구문 수행
7.  적절한 부모 클래스의 인스턴스 생성자 수행
8.  인스턴스 생성자 수행

> 클래스 자체에 대한 초기화 작업(1~4)는 단 한 번만 이루어진다. 즉, 추가 인스턴스가 생성되면 5단계부터 수행된다.
