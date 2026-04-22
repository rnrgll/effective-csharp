## 21. 타입 매개변수가 `IDisposable`을 구현한 경우를 대비하여 제너릭 클래스를 작성하라
### 타입 매개변수가 `IDisposable`을 구현한 경우 발생할 수 있는 문제

* 제네릭 제약 조건(`where T : ...`)은 타입이 반드시 제공해야 하는 기능만 규정한다.

* 하지만 타입 매개변수로 지정하는 타입이 **`IDisposable`을 구현하고 있다면 특별한 추가 작업이 반드시 필요**하다.

  만약 제네릭 클래스 내부에서 생성한 `T`가 비관리 자원(`IDisposable`)을 사용한다면, 개발자가 명시적으로 처리하지 않을 경우 **메모리 누수(Memory Leak)**가 발생할 수 있기 때문이다.

* 예시

  ```csharp
  public interface IEngine
  {
  	void DoWork();
  }
  
  public class EngineDriverOne<T> wher T : IEngine, new()
  {
  	public void GetThingsDone()
  	{
  		T driver = new T();
  		driver.DoWork();
  	}
  }
  ```

  : `driver`가 `IDisposable`을 구현하고 있는 경우 메모리 누수가 발생한다. 🚨



### 상황별 해결 방법

* 타입 매개변수가 지역 변수일 때

  * `using`문을 통해 해제할 수 있다.

  * 컴파일러는 `IDisposable`로 형변환된 객체를 저장하기 위해 숨겨진 지역변수를 생성하고, 지역 변수의 `null` 여부에 따라 `Dispose`의 호출 여부가 결정된다.

    * `T`가 `IDisposable`을 구현하지 않은 경우, `null`을 반환하며 아무 동작도 하지 않는다.

    * `T`가 `IDisposable`을 구현한 경우, 자동으로 `Dispose()`를 호출한다.

  * 예시

    ```csharp
    public void GetThingsDone()
    {
        T driver = new T();
        // driver가 IDisposable이면 Dispose() 호출, 아니면 무시
        using (driver as IDisposable) 
        {
            driver.DoWork();
        }
    }
    ```

* 타입 매개변수가 멤버 변수일 때

  * 제너릭 클래스에서 `IDisposable` 인터페이스를 구현해 리소스를 처리해야 한다.

    ```csharp
    public sealed class EngineDriver2<T> : IDisposable where T : IEngine, new()
    {
    	// 생성 작업이 오래걸릴 수도 있으므로, Lazy를 이용하여 초기화
    	private Lazy<T> driver = new Lazy<T>(() => new T());
    
    	public void GetThingsDone() => driver.Value.DoWork();
    
    	// IDisposable 멤버
    	public void Dispose()
    	{
    		if (driver.IsValueCreated)
    		{
    			var resource = driver.Value as IDisposable;
    			resource?.Dispose();
    		}
    	}
    }
    ```

    <figure><p><code>sealed</code> 클래스라면 제너릭 클래스에 대한 <code>Dispose</code> 함수만 구현하면 되지만, 상속될 가능성이 있다면 ‘아이템 17’의 표준 <code>Dispose</code> 패턴 전체를 구현해야 한다.</p></figure>

  * 제너릭 클래스 구현이 너무 복잡하다면, 객체의 소유권(생성 및 해제) 자체를 외부에 위임하고, 제너릭 클래스에서는 이를 주입받아 사용하는 형태로도 구현할 수 있다.

    ```csharp
    public sealed class EngineDriver<T> where T : IEngine
    {
        private T driver;
    
        public EngineDriver(T driver)
        {
            this.driver = driver;
        }
    
        public void GetThingsDone()
        {
            driver.DoWork();
        }
    }
    
    ```

    <figure><p>👨‍🏫 가장 보편적인 패턴이다.</p></figure>
