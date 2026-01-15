
# 17\. 표준 Dispose 패턴을 구현하라

## 17.1 Dispose 패턴

.NET에서는 비관리 리소스를 안전하게 정리하기 위해 **표준화된 Dispose 패턴**을 제공한다.

**`IDisposable` 인터페이스**를 통해 리소스를 누수를 방지한다.

비관리 리소스에 대한 명시적 정리가 없는 경우에도 `finalizer` 를 통해 리소스가 정리되도록 한다.

### 17.1.1 베이스 클래스에서 해야 할 일

비관리 리소스를 직접 포함하는 베이스 클래스는 다음을 구현해야 한다.

*   리소스를 정리하기 위해서 **`IDisposable 인터페이스`를 구현**해야 한다.
*   **방어적으로 동작할 수 있도록 `finalizer`를 추가**해야 한다.
    *   `Dispose()` 메서드를 호출되지 않을 가능성을 고려한다.
    *   비관리 리소스가 어떠한 경우에도 누수되지 않도록, 올바르게 정리될 수 있도록 `finalizer` 를 구현해야 한다.
*   `Dispose` 와 `finalizer` 에서 실제 리소스 정리 작업을 **다른 가상 메서드(`Dispose(bool)`)에 위임한다.**
    *   이를 통해 파생 클래스가 고유의 리소스 정리 작업이 필요한 경우 **가상 메서드를 재정의**할 수 있다.

### 17.1.2 파생 클래스에서의 해야 할 일

*   파생 클래스가 고유의 리소스 정리 작업을 수행해야 한다면 **베이스 클래스에서의 가상 메서드(`Dispose(bool)`)를 재정의**한다.
*   멤버 필드로 비관리 리소스를 포함하는 경우에만 `finalizer` 를 추가한다.
*   베이스 클래스에서 정의하고 있는 **가상 함수를 반드시 재호출해야 한다.**

## 17.2 IDisposable Interface

`Dispose()` 메서드 하나만을 가진다.

```csharp
public interface IDisposable
{
	void Dispose();
}
```

### **Dispose()에서 반드시 수행해야 할 작업 4가지**

1.  모든 비관리 리소스 정리
    
2.  모든 관리 리소스 정리
    
3.  객체가 이미 정리되었음을 나타내기 위한 상태 플래그 설정
    
    → 이미 정리된 객체에 대해 추가로 정리 작업이 요청될 경우 플래그를 확인하고 `ObjectDisposed` 예외를 발생
    
4.  `finalizer` 호출 억제
    
    → `GC.SuppressFinalizer(this)` 를 호출하여 finalizer 큐에 들어가는 비용을 제거할 수 있다.
    

```csharp
public void Dispose()
{
    // 비관리 리소스 정리
    Dispose(true);
    // finalizer 호출 억제
    GC.SuppressFinalize(this);
}
```

## 17.3 `Dispose(bool)` : 가상 헬퍼 함수

### 17.3.1 필요한 이유

*   finalize와 Dispose() 메서드가 일반적으로 동일한 역할을 하므로 중복된 코드가 여러 번 나타날 수 있음
*   상속 구조에서 베이스/파생 클래스가 각자 리소스를 가지며, 중복 코드를 피하면서 파생 클래스 확장을 허용해야 한다.

### 17.3.2 가상 헬퍼 함수

이러한 문제를 해결하기 위해 가상 헬퍼 함수를 사용한다.

```csharp
protected virtual void Dispose(bool isDisposing);
```

*   가상 헬퍼 함수는 리소스 정리를 위한 공통 작업을 수행하고 파생 클래스에게 리소스 정리를 할 수 있도록 제공한다.
*   파생 클래스에서 베이스 클래스의 리소스를 해제할 수 있다.
*   **Dispose()나 finalize를 구현할 때 가상 함수를 이용할 수 있다.**
*   파생 클래스의 `Dispose(bool)` 코드의 마지막 부분에서 **반드시 <span class="fontColorRed">베이스 클래스에서 정의하고 있는 <code>Dispose(bool)</code> 함수를 호출</span>해야 한다.**
*   `isDisposing`
    *   true : 관리 리소스와 비관리 리소스 모두 제거
    *   false : 비관리 리소스만 제거

### 17.3.3 예시 코드

#### 베이스 클래스

```csharp
using System;
 
class BaseClass : IDisposable
{
    private bool _disposed = false; // 중복 처리 방지
 
    ~BaseClass() => Dispose(false);  // finalizer : Dispose(bool) 메서드를 통해 중복 코드 작성 X
 
    public void Dispose()
    {
        Dispose(true); // 실제 정리 수행은 가상 메서드로 위임
        GC.SuppressFinalize(this); // finalizer 회피
    }
 
    protected virtual void Dispose(bool disposing)
    {
        if (_disposed) // 이미 정리된 경우
        {
            return;
        }
 
        if (disposing)
        {
            // todo : 관리 리소스 정리
        }
 
		// todo: 비관리 리소스 정리

        _disposed = true; 				// 플래그 설정
    }
}

```

#### 파생 클래스

```csharp
using System;
 
class DerivedClass : BaseClass
{
    bool _disposed = false;   // 파생 클래스 혹은 베이스 클래스 일부만 정리된 경우가 있을 수도 있으므로 플래그를 이중으로 배치
 
    ~DerivedClass() => Dispose(false);
 
    protected override void Dispose(bool disposing)
    {
        if (_disposed)
        {
            return;
        }
 
        if (disposing)
        {
            // TODO: 관리 리소스 정리
        }
 
        // TODO: 비관리 리소스 정리
        _disposed = true;
 
        base.Dispose(disposing);
    }
}

```

## 17.4. Dispose 패턴 구현시 주의사항

1.  **Dispose와 finalizer는 방어적으로 작성되어야 한다.**
    
    *   `Dispose()`는 여러 번 호출될 수 있다.
    *   일부 리소스만 정리된 상태에서도 다시 호출되어도 안전해야 한다.
    *   Dispose된 객체는 더 이상 유효하지 않다. 접근 시 `ObjectDisposedException` 예외 발생시키는 것이 표준이다.
2.  **이미 정리된 객체의 필드에 접근하지 않아야 한다.**
    
    *   Dispose된 객체는 더 이상 유효하지 않다.
    *   이미 Disposed 된 객체 혹은 초기화가 제대로 완료되지 않은 객체에 대해서도 finalizer가 호출될 수 있다.
        *   Dispose가 구현이 잘못되어 `GC.SuppressFinalize`를 호출하지 않은 경우
        *   상속 구조에서 `base.Dispose()`가 누락되는 등의 상황
    *   Finalizer는 객체가 ‘정상 상태’라고 보장되지 않는 시점에도 호출될 수 있기 때문에, Finalizer 안에서는 객체의 다른 필드 상태를 절대 믿으면 안 된다.
3.  **비관리 리소스를 직접 포함하지 않는 경우라면 finalizer를 구현하지 말아야 한다.**
    
    *   finalizer는 성능 비용이 크다.
    *   비관리 리소스가 없는 타입에는 필요 없다.
4.  **Dispose 메서드 내에서는 리소스 정리 작업만 수행해야 한다.**
    
    *   Dispose나 finalizer에서 <span class="fontColorRed"><strong>다른 작업을 수행하게 되면 객체의 생명주기가 깨진다</strong>.</span>
    *   특히 finalizer에서 객체를 다시 참조 가능하게 만들면 **<span class="fontColorRed">객체가 다시 살아나는 문제</span>가 발생한다.**
    
    ```csharp
    public class BadClass
    {
    	private static readonly List<BadClass> finalizedList = new List<BadClass>();
    	private string msg;
    	
    	public BadClass(string msg) { msg = (string) msg.Clone(); }
    	~BadClass() { finalizedList.Add(this); }
    	// finalizer를 실행할 때 전역 목록에 자신을 추가하기에 객체는 다시 살아난다.
    }
    ```
    
    *   되살아난 객체는 정상적인 상태가 아니며, **GC는 이미 finalizer를 호출했다고 판단하기 때문에 <span class="fontColorRed">해당 객체의 finalizer를 다시 호출하지 않는다.</span>**
        
        ⇒ 객체를 정리하려고 해도 다시 finalizer를 호출할 방법이 없다!
