# 49\. **catch 후 예외를 다시 발생시키는 것보다 예외 필터가 낫다**

## **49.1 표준 catch절의 예외 처리**

표준적인 `catch`는 다음 방식으로 동작한다.

*   예외 타입에 따라 그에 부합하는 예외만을 처리한다.
*   예외를 우선 잡은 후 분석과정 수행한다.
*   복구 불가능하다고 판단되면 다시 `throw` 로 예외를 전파한다.
    *   해당 try/catch 묶음의 다른 catch로 들어가지 않고 해당 try 블록 밖으로 예외를 던진다. 다음 처리는 **상위 호출자**에서 찾는다.
*   또는 해당 블록에서 처리할 예외가 아니라면 다시 예외를 던지는 코드를 넣는다.

```csharp
var retryCount = 0;
var dataString = default(String);

 
while (dataString == null)

{
     try
     {
          dataString = MakeWebRequest();
     }
     catch (TimeoutException e)
     {

           if (retryCount++ < 3)
           {
                //재시도 이전에 잠깐 멈춤
                Task.Delay(1000 * retryCount);
           }
           else
               throw;  // StackTrace 유지. 원래 TimeoutException을 재던짐


      }
}
```

### 49.1.1 스택 되감기(stack unwinding)

예외는 그 자리에서 처리하지 못하면 **호출자를 따라 위로 전파**된다.

이때 런타임은 단순히 위로 전달만 하는게 아니라, **예외가 지나가는 스택 프레임을 정리하는데, 이 과정을 스택 되감기라고 한다.**

되감기 과정 예시

1.  메서드 A → B → C 호출 중에 C에서 예외가 발생
2.  C에서 처리 못 함
3.  런타임은 C의 스택 프레임을 정리(B로 이동하기 위해)
4.  B에서도 처리 못 하면 B 프레임도 정리(A로 이동)

이런 식으로 스택 되감기가 진행된다.

스택 프레임 정리에서 일어나는 일

*   해당 프레임에서 `finally`가 있으면 실행된다.
    
*   **지역 변수는 그 프레임과 같이 사라진다. ⇒ 예외가 발생한 정확한 시점의 지역 변수 상태에 접근하기 어려워진다.**
    

> #### <p>지역 변수가 클로저에 의해 캡처된 경우</p>
> 지역 변수가 클로저에 의해 캡처된 경우에는 여전히 접근 가능하다.
> 어떤 지역 변수가 람다/로컬 함수 같은 곳에서 사용되면, 컴파일러가 그 변수를 스택이 아니라 힙(객체) 쪽으로 빼서 저장하는 경우가 있다.
> 이렇게 ‘밖’에서 참조 가능한 상태로 끌어올려진 변수는 스택 프레임이 사라져도 남아 있을 수 있다. 이를 ‘캡처(capture) 되었다고 한다.
> 
> ```C#
> int retryCount = 0;
> Func<bool> canRetry = () => retryCount++ < 3;
> // retryCount는 람다에서 사용되므로 캡처될 수 있음
> ```
> <code>retryCount</code> 는 단순 지역 변수가 아니라, 컴파일러가 만든 ‘캡처 클래스’ 안의 필드 처럼 취급될 수 있고, 그래서 스택 프레임이 사라져도 값이 남아 있을 수 있다.

### 49.1.2 문제점

*   코드 분석이 어렵다. (catch 안에서의 조건 분기 + 다시 throw)
    
*   **추가적인 런타임 비용이 발생한다.**
    
    *   **catch 진입에서 비용이 발생 :** 예외 처리 흐름으로 전환, 스택 프레임 정리, finally 실행 등
    *   catch 로 들어갔다가 다시 throw 하게 되면 위에서 또 다시 처리를 해야하므로 추가 비용이 발생한다.
*   catch가 선택되는 순간 런타임은 **스택 되감기(stack unwinding)** 를 수행되어
    
    *   예외가 발생한 메서드의 스택 프레임이 정리 → 그 메서드 내부 지역 변수들에 접근할 수 없다.
    *   상태 정보의 일부가 손실된다.
*   다시 `throw` 하게 되면, **예외 발생 위치가 바뀌게 됨**
    
    → 어디서 예외가 발생했는지, 왜 예외가 발생했는지를 디버깅하는데 어려워짐
    

> #### <p><code>throw;</code> VS <code>throw e;</code></p>
> ```C#
> throw;     // 원래 예외를 "그대로 다시 던짐" (스택트레이스 보존)
> throw e;   // 예외를 "다시 던지는 척 새로 던짐" (스택트레이스 초기화)
> ```
> <ul><li>throw;
> <ul><li>원래 예외 객체를 그대로 던짐</li>
> <li>예외가 처음 발생한 위치의 StackTrace 유지</li>
> <li>원인이 어디서 터졌는지 그대로 남음</li></ul></li>
> <li>throw e;
> <ul><li>같은 예외 객체를 던지긴 하지만</li>
> <li>StackTrace가 현재 catch 지점부터 새로 시작</li>
> <li>실제 원인이 사라짐</li></ul></li></ul>

## 49.2 예외 필터 (Exception Filter)

예외 필터는 “catch로 들어가기 전에” 예외를 잡을지 말지를 결정하는 방법이다.

```csharp
catch (SomeException e) when (조건)
{
    // 조건이 true일 때만 catch로 진입
}
```

### 49.2.1 특징

*   예외 필터는 스택이 되감기 전에 평가된다. **즉, 예외가 발생한 메서드 프레임이 정리되기 전에 조건을 검사한다. ⇒ 호출 스택이나 지역변수에 대한 모든 정보에 접근할 수 있다.**
*   필터 조건이 false면, **해당 catch는 ‘없었던 것처럼’ 취급된다.** 런타임은 계속 콜 스택을 따라 올라가면서 적절한 catch 를 찾는다.
*   필터가 false일 때는 catch로 **‘진입’하지 않았기 때문에** 스택 되감기, finally 실행, 지역 변수 해제 같은 비용이 비싼 작업들이 일어나지 않는다. ⇒ **프로그램 상태가 ‘원형’에 가깝게 유지된다.**
*   기존 catch 방식처럼 예외를 일단 잡고 → 다시 던지는 구조를 줄일 수 있다.

### 49.2.2 예시 코드

```csharp
var retryCount = 0;
var dataString = default(String);

while (dataString == null)
{
     try
     {
         dataString = MakeWebRequest();   //호출 스택이 여기로 찍힘
     }
     catch (TimeoutException e) when (retryCount++ < 3)
     {

           if (retryCount < 3)
           {
                //재시도 이전에 잠깐 멈춤
                Task.Delay(1000 * retryCount);

           }

      }

}
```

*   TimeoutException이 나더라도
    *   `retryCount < 3`이면 catch로 들어가서 재시도 로직 수행
    *   `etryCount >= 3`이면 필터가 false가 되어 이 catch는 적용되지 않고,

### **49.2.3 예외 필터의 장점**

*   catch로 진입하기 전에 조건을 판단하므로, 스택 되감기 이전의 상태에서 디버깅/분석이 가능하다.
*   예외가 발생한 지점의 호출 정보(호출 스택) 및 지역 변수 상태가 더 잘 보전된다.
*   필터가 false일 때는 catch로 진입하지 않으므로 불필요한 스택 되감기나 finally 실행이 줄어들 수 있고, 결과적으로 성능이 개선될 수 있다.
*   ‘처리할 예외만 처리’하고, 처리하지 않을 예외는 자연스럽게 상위로 흘려보낼 수 있다.

## 49.3 예외 필터가 효과가 큰 상황들

예외를 잡아서 내부를 확인해보고 다시 던지는 패턴이 자주 나오는 곳에서 효과가 크다.

### 1) Task / 비동기에서 AggregateException
여러 자식 Task가 동시에 실패하면, 예외가 하나로 합쳐져 AggregateException으로 들어오는 경우가 있다.

이때 모든 예외를 무조건 catch로 받아 처리하기보다, 필요한 경우만 처리하고 싶은 경우 필터로 특정 예외가 포함된 경우에만 처리하도록 할 수 있다.

```C#
try
{
    await Task.WhenAll(tasks);
}
catch (AggregateException ex) when (ex.InnerExceptions.Any(e => e is TimeoutException))
{
    // TimeoutException이 섞여 있는 경우에만 여기서 처리(예: 재시도)
    RetryAll();
}

```
### 2) COMException과 HResult 기반 분기
<code>COMException</code> 클래스는 HResult 속성을 갖고 있다. 이 값은 COM 객체 호출에 의해 반환되는 는 HRESULT 코드를 그대로 담고 있으며, 같은 <code>COMException</code>이라도 <code>HResult</code> 값에 따라 의미가 완전히 달라질 수 있다.

<ul><li>어떤 HRESULT는 잠깐 기다렸다 재시도하면 성공할 수 있는 오류이고</li>
<li>어떤 HRESULT는 즉시 실패로 처리해야 하는 오류일 수 있다.</li></ul>

이렇게 내부 속성(HResult) 값을 기준으로 예외 처리를 해야하는 경우 필터를 사용하면 된다. 처리할 가치가 있는 COM 예외만 선별적으로 처리할 수 있다.

```C#
const int RPC_E_SERVERCALL_RETRYLATER = unchecked((int)0x8001010A);

try
{
    CallComObject();
}
catch (COMException ex) when (ex.HResult == RPC_E_SERVERCALL_RETRYLATER)
{
    // '잠깐 기다렸다 재시도'가 가능한 HRESULT인 경우만 처리
    Thread.Sleep(200);
    CallComObject();
}


```

### 3) HTTPException / HTTP 상태 코드 기반 분기
HTTP 요청의 실패시 응답 코드에 따라 여러 방식으로 처리해야 한다.

​​​​​복구할 수 있는 실패와 그냥 실패로 흘려보내야 하는 경우를 구분해서 처리해야하는데, 예외 필터를 사용하면 자연스럽게 분기 처리할 수 있다.

```C#
try
{
    var response = await client.GetAsync(url);
    response.EnsureSuccessStatusCode(); // 실패면 HttpRequestException
}
catch (HttpRequestException ex) when (TryGetStatusCode(ex, out var code) && (code == 301 || code == 302))
{
    // redirect인 경우만 여기서 처리
    url = GetRedirectUrlSomehow();
    // 다시 시도
}
```

- HTTP 요청이 실패하면 HttpRequestException이 발생한다.
- 예외 필터에서 상태 코드를 확인하고, 상태 코드가 301 또는 302인 경우에만 catch로 진입하여 리다이렉트 처리 및 재시도를 수행한다.
