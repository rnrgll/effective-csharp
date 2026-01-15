# 12\. 할당 구문보다 멤버 초기화 구분이 좋다

생성자 내부에서 멤버 변수 초기화 코드를 작성하다 보면, 특정 생성자에서 초기화가 누락되는 문제가 발생할 수 있다.

이를 방지하기 위해서 **생성자 본문에서 멤버 변수에 값을 할당하기 보다 멤버 초기화 구문을 사용하는 것이 더 안전하다.**

## 12.1 멤버 초기화 구문

```csharp
public class MyClass
{
	// 멤버 변수를 선언할 때 객체를 함께 생성하여 초기화
	private List<string> labels = new List<string>();
}
```

### 특징

*   모든 생성자에서 공통적으로 초기화가 보장된다.
    *   컴파일러가 **모든 생성자의 시작 부분에서 멤버 초기화 구문을 자동으로 삽입한다.**
    *   기본 생성자가 암시적으로 생성되는 경우에도 동일하게 동작한다.
*   **부모 클래스의 멤버 초기화가 자식 클래스보다 먼저 수행된다.**
*   코드 유지보수에 유리하다.
    *   생성자가 여러 개일 경우에도 초기화 로직을 한 곳에서 관리할 수 있다.
    *   초기화 누락으로 인한 버그를 예방할 수 있다.

## 12.2 멤버 초기화 구문을 사용하지 않는 것이 좋은 경우

1.  **기본값(0, null)로 초기화 되는 경우**
    
    시스템 초기화 과정에서 **필드는 기본적으로** `default` 값으로 초기화된다. 따라서 명시적으로 0 또는 null로 다시 초기화할 필요가 없다. 오히려 **중복 코드가 생성된다.**
    
    ```csharp
    public struct MyValType  {}
    
    MyValType myVal1; // default로 초기화
    MyValType myVal2 = new MyValType(); // default를 또 한 번 명시적으로 초기화 (의미상 중복)
    ```
    
    → 의미상 차이가 없으며, 가독성과 효율 측면에서 좋지 않다.
    
2.  **생성자마다 서로 다른 방식으로 객체를 초기화해야 하는 경우**
    
    객체 생성 방식이 생성자마다 다를 경우, 멤버 초기화 구문을 사용하면 **오히려 불필요한 중복 객체 생성이 발생한다.**
    
    ```csharp
    public class MyClass2
    {
    	private List<string> labels = new List<string>();
    	
    	MyClass2() 
    	{
    	}
    	
    	MyClass2(int size) 
    	{
    		labels = new List<string>(size);
    	}
    }
    ```
    
    이 코드는 실제로 아래와 같은 흐름이 된다.
    
    ```csharp
    MyClass2(int size)
    {
        labels = new List<string>();        // 멤버 초기화 구문
        labels = new List<string>(size);    // 생성자 본문
    }
    
    ```
    
    *   멤버 초기화 구문에서 List 객체가 한 번 생성된다.
    *   생성자 본문에서 바로 다시 List 객체를 생성하면서, **앞서 생성된 객체는 즉시 가비지가 된다.**

    > <p>**자동 속성(Property) 초기화에도 비슷한 문제가 발생할 수 있다.**</p>
    > C#에서는 프로퍼티를 선언과 동시에 초기화할 수 있는데, 이 경우에도 불필요한 객체 생성이 포함될 수 있다.
    > 
    > ```c#
    > public class MyClass
    > {
    >    public List<string> Labels { get; set; } = new List<string>();
    >    		
    >    public MyClass()
    >    {
    >    }
    >    		
    >    public MyClass(int size)
    >    {
    >       Labels = new List&lt;string&gt;(size);
    >    }
    > }
    > ```
    > 자동 속성 초기화도 멤버 초기화 구문과 동일하게 동작한다.
    > <strong>컴파일러가 생성자 앞부분에 초기화 코드를 삽입한다.</strong>
    > 위 코드는 실제로 아래와 같은 흐름으로 실행된다.
    >
    > ```C#
    > public MyClass(int size)
    > {
    >     // 자동 속성 초기화 (컴파일러가 자동 삽입)
    >     Labels = new List&lt;string&gt;();
    > 
    >     // 생성자 본문
    >     Labels = new List&lt;string&gt;(size);
    > }
    > ```
    > 즉, List 객체가 두 번 생성되며, 첫 번째 List는 즉시 가비지가 된다.
        
3.  **예외 처리가 반드시 필요한 경우**
    
    멤버 초기화 구문은 `try`로 감쌀 수 없다. 따라서 초기화 중 **발생한 예외는 그대로 외부로 전파되고 클래스 내부에서 복구를 시도할 수 없다.**
    
    예외 처리가 반드시 필요한 경우에는 멤버 초기화 구문 대신 **생성자 내부에서 초기화 + 예외 처리**를 해야한다.

    
    ```csharp
    private Resource res = new Resource(); // 예외 발생 시 처리 불가
    
    public MyClass()
    {
        try
        {
            res = new Resource();
        }
        catch (Exception e)
        {
            // 예외 처리 또는 복구 로직
        }
    }
    ```
