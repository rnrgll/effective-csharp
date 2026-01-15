# 15\. 불필요한 객체를 만들지 말라

## 15.1 GC와 성능

*   객체를 생성 및 해제는 **상대적으로 많은 CPU 비용(많은 프로세서 시간)을 사용한다.**
*   객체 생성과 해제가 빈번하게 발생하면 **GC가 자주 수행**되고, 이는 심각한 성능 저하로 이어질 수 있다.
*   따라서 GC가 과도하게 동작하지 않도록 **불필요한 객체 생성을 줄이는 것**이 중요하다.

## 15.2. GC의 부담을 줄이는 법

### 15.2.1 자주 사용되는 지역 변수를 멤버 변수로 변경하기

지역 변수는 메서드 실행이 끝나면 더 이상 참조되지 않아 **곧바로 가비지가 된다**.

자주 호출되는 메서드에서 지역 변수를 매번 새로 생성하면

*   메모리 할당과 해제 반복적으로 발생
*   이로 인해 GC가 더 자주 실행

```csharp
protected override void OnPaint(PaintEventArgs e)
{
	using(Font MyGont = new Font("a", 10.0f)) 
	{
			e.Graphics.DrawString(DataTime.Now.ToString(), **MyFont**, Brushes.Black, new PointF(0,0));
			// ....
	}
}
```

위의 경우 `OnPaint`가 호출될 때마다 `Font` 객체를 매번 재생성한다.

이를 멤버 변수로 한 번만 생성해 재사용하면 불필요한 객체 생성을 줄일 수 있다.

```csharp
Font myFont =  = new Font("a", 10.0f);

protected override void OnPaint(PaintEventArgs e)
{
		e.Graphics.DrawString(DataTime.Now.ToString(), myFont, Brushes.Black, new PointF(0,0));
		//....
}
```

### **15.2.2 자주 사용되는 참조 타입을 정적 멤버로 공유하기**

멤버 변수로 객체를 옮기더라도, **인스턴스마다 동일한 객체가 생성된다면 여전히 낭비**가 발생할 수 있다.

이처럼 여러 인스턴스에서 **공통으로 사용되는 객체는 정적 멤버로 선언해 하나의 인스턴스를 공유**하는 것이 효율적이다.

위의 예시 코드에서 `Brushes.Black`이 이에 해당한다.

```csharp
private static readonly Brush BlackBrush = Brushes.Black;
```

### \+ 지연 초기화 적용하기

정적 멤버는 타입이 초기화 된 이후 프로그램 종료 시까지 메모리에 유지된다.

실제로 사용되지 않더라도, 타입 초기화 시점에 정적 멤버가 초기화되어 객체가 생성되면 불필요한 메모리 점유가 발생할 수 있다.

이러한 문제를 완화하기 위해서 객체가 실제로 필요해지는 시점까지 생성을 미루는 **지연 초기화(Lazy Initializtion)을 적용한다.**

### **15.2.3 종속성 주입(DI)을 통한 객체 재사용 활용하기**

자주 사용되는 객체를 외부에서 한 번만 생성한 뒤 필요한 곳에 주입받아 재사용하는 방식도 효과적이다.

```csharp
 class Service
{
}

class Consumer
{
    private readonly Service _service;

    public Consumer(Service service)
    {
        _service = service;
    }
}
```

### **15.2.4 변경 불가능(Immutable) 타입 사용 시 주의하기**

#### 변경 불가능한 타입이란?

*   한 번 생성되면 **내부 상태를 변경할 수 없는 타입**
    
*   값이 변경되는 것처럼 보여도 **실제로는 새로운 객체가 생성된다**
    
*   예) `System.String`
    
    문자열을 연결할 때마다 **새로운 string 객체가 생성되고 이전 객체는 가비지가 된다.** 이러한 작업이 반복되면 GC 부담이 크게 증가한다.
    

#### 해결 방법

1.  문자열 보간 사용하기
    
    단순한 문자열 결합의 경우, 문자열 보간을 사용하면 된다.
    
    ```csharp
    string s = $"{a} {b}";
    ```
    
2.  `StringBuilder` 사용하기
    
    *   `StringBuilder`는 가변 문자열을 다루기 위한 타입이다.
    *   내부적으로 **버퍼(char 배열)를 유지**하여, 문자열을 수정할 때 새 객체를 계속 만들지 않고, 기존 버퍼를 재사용한다.
    
    ```csharp
    var sb = new StringBuilder();
    sb.Append("Hello");
    sb.Append(" ");
    sb.Append("World");
    
    string result = sb.ToString();
    ```
    
    *   문자열을 여러 단계에 걸쳐 구성해야하는 경우, 반복적인 문자열 결합이 필요한 경우에 적합하다.


> #### <p>❗설계 관점에서의 포인트</p>
> 불변 객체를 설계할 때는 <code>string</code> 과 <code>StringBuilder</code> 과 같이 <strong>객체를 단계적으로 구성할 수 있는 보조 수단을 함께 제공하는 것이 중요하다.</strong> 객체의 불변성을 유지하면서도 성능과 사용성을 모두 확보할 수 있다.
