# 50.**예외 필터의 다른 활용 예를 살펴보라**

## **50.1 활용1 : 로깅**

예외 필터를 이용하면 예외를 실제로 처리(catch 진입)하지는 않되, 예외가 발생한 순간에 **로깅만 수행**하도록 만들 수 있다.

<span class="fontColorRed"><code>catch (Exception e)</code>는 모든 예외를 대상으로 잡을 수 있다</span>. 이때 <span class="fontColorRed"><code>when</code> 필터가 <strong>항상 false</strong></span>를 반환하게 만들면

*   catch 블록으로 **진입하지 않는다.**
*   대신 필터 식(메서드 호출)을 통해 **로깅만 수행**한다
*   이후 런타임은 **같은 try 블록의 다음 catch를 계속 찾거나**, 없으면 **콜 스택 위로 예외를 전파**한다.

→ 이렇게 필터 평가 시점에 로그만 남기고 예외는 원래 흐름대로 보낼 수 있게 된다.

→ 예외 흐름을 바꾸지 않기 때문에 호출 스택을 망치지도 않는다.

### 예시 1) 모든 예외 로깅

```csharp
public static bool ConsoleLogException(Exception e)
{
    var oldColor = Console.ForegroundColor;
    Console.ForegroundColor = ConsoleColor.Red;
    Console.WriteLine($"Error: {e}");
    Console.ForegroundColor = oldColor;

    return false; // 무조건 false → catch로 진입하지 않게 함
}

try
{
    data = MakeWebRequest();
}
catch (Exception e) when (ConsoleLogException(e))
{
    // 절대 진입하지 않음 (필터가 false이므로)
}
catch (TimeoutException e) when (failures++ < 10)
{
    Console.WriteLine("Timeout error : trying again");
}

```

*   예외가 발생하면 첫 번째 catch의 when이 실행되며 로그가 찍힌다
*   when이 false이므로 첫 번째 catch는 적용되지 않는다
*   그 다음 catch(TimeoutException)를 계속 검사한다
*   TimeoutException이고 조건을 만족하면 여기서 처리된다

### 예시 2) 특정 예외만 로깅하기

로깅용 catch 타입을 바꾸어 특정 예외에 대해서만 로그를 찍을 수 있다.

```csharp
catch (IOException e) when (ConsoleLogException(e))
{
}
```

### 예시 3) 특정 예외를 제외하고 로깅하기

로깅용 catch 순서를 바꾸어 특정 예외 상황을 제외하고 로깅을 할 수 있다.

```csharp
try
{
    data = MakeWebRequest();
}
catch (TimeoutException e) when (failures++ < 10) // 이 예외는 로깅하지 않음
{
    Console.WriteLine("Timeout error : trying again");
}
catch (Exception e) when (ConsoleLogException(e))
{
}

```

*   TimeoutException은 위에서 잡혀서 처리되고,
*   그 아래의 로깅 필터 catch까지 도달하지 않으므로 로그가 찍히지 않는다.

## 50.2 활용 2 : 디버깅

예외 필터는 디버깅에서도 쓸 수 있다. 디버거가 붙어있을 때는 예외를 처리하지 말고 예외 필터를 통과하게 하여 디버거가 즉각적으로 예외를 감지할 수 있게 할 수 있다.

```csharp
try
{
    data = MakeWebRequest();
}
catch (Exception e) when (ConsoleLogException(e))
{
}
catch (TimeoutException e) when ((failures++ < 10) && (!System.Diagnostics.Debugger.IsAttached))
{
    Console.WriteLine("Timeout error : trying again");
}

```

*   디버거가 없는 일반적인 상황에서는 TimeoutException이 나면 재시도를 한다.
    
*   디버거가 있는 상황에서는 필터 조건을 false로 만들어 catch에 진입하지 않게 한다. (재시도 루틴이 예외를 삼켜버리면 원인 추적이 어려움)
    
    → 그러면 예외가 상위로 전파되면서 디버거가 즉시 예외를 감지한다.
    
    → 예외 발생 시점에서 바로 멈추면서 원인 추적이 쉬워진다.
