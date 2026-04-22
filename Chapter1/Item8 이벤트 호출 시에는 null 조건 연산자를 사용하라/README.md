## 8. 이벤트 호출 시에는 `null` 조건 연산자를 사용하라

### 문제 상황

* C#의 이벤트(델리게이트)는 이벤트 핸들러(구독자)가 없을 때 `null` 상태를 진다.

* 따라서 이벤트 핸들러가 없는 이벤트를 호출(Invoke)하려고 하면, **`NullReferenceException`** 오류가 발생한다.

* 예시

  ```csharp
  public event EventHandler Updated;
  
  public void RaiseUpdates()
  {
      // 만약 구독자가 아무도 없다면 Updated는 null임
      Updated(this, EventArgs.Empty); // 🚨 예외 발생!
  }
  
  ```

### 과거의 해결 방법들과 문제점

* 방법 1. 실행 전 null 체크

  * `if`문으로 null 여부를 확인하는 방법

    ```csharp
    if (Updated != null)
    {
        Updated(this, EventArgs.Empty);
    }
    
    ```

  * 문제: 멀티 스레드 환경에서 Race Condition이 발생할 수 있다.

    * `if (Updated != null)`을 통과한 직후, 다른 스레드가 이벤트 구독을 취소해버리면 `Updated(this, ...)`를 실행하는 순간에는 `null`이 되어 예외가 발생한다.

* 방법 2. 이벤트를 로컬 변수에 얕은 복사

  * 이벤트를 로컬 변수에 할당(얕은 복사)한 후, 복사된 변수로 이벤트를 호출하는 방법

    ```csharp
    var handler = Updated; // 1. 현재 상태를 로컬 변수에 캡처 (얕은 복사)
    if (handler != null)
    {
        handler(this, EventArgs.Empty); // 2. 원본 Updated가 null이 되어도 handler는 살아있음
    }
    
    ```

  * 장점: 스레드 안정성이 확보된다.

  * 문제점: 코드의 가독성이 떨어진다.

 

### 해결 방법: null 조건 연산자(`?.`)

* null 조건 연산자 (`?.`)

  * C# 6.0부터 도입된 문법

  * 동작 방식

    ```csharp
    obj?.field
    obj?.Method()
    
    ```

    * obj가 null이면 아무 것도 하지 않고 null 반환

    * obj가 null이 아니면 필드 정상 접근 / 메서드 정상 호출

* 동작 방식

  ```csharp
  public void RaiseUpdates()
  {
      Updated?.Invoke(this, counter);
  }
  
  ```

  * 이벤트가 `null` 상태면 아무 동작도 하지 않고 넘어갑니다.

  * 이벤트가 `null` 상태가 아니면 `Invoke` 메서드를 실행합니다.

* 장점

  * 스레드 안전성과 가독성 문제가 해결된다.

    <figure><p>📝 이벤트가 <code>null</code> 상태인지 아닌지 평가하는 과정에 원자적으로 수행된다.</p></figure>

* 주의 사항

  * 일반적인 메서드 호출 문법을 지원하지 않기 때문에, 이벤트(델리게이트)가 내부적으로 가지고 있는 `Invoke` 메서드를 명시적으로 호출해야 한다.



</br>

> 👨‍🏫 이벤트처럼 대상의 존재 여부를 보장할 수 없는 상황에서는 null 조건 연산자를 사용해 안전하게 처리하는 것이 좋다.</br>
> 그러나 모든 곳에서 습관적으로 사용해서는 안된다!</br>
> 반드시 있어야하는 중요한 객체에도 습관적으로 null 조건 연산자를 사용하면, 오류가 발생했을 때 에러 메시지 없이 무시되어 버릴 수 있다.</br>
> -> 나중에 작동이 안되는 원인을 찾기 힘들다</br>

 
