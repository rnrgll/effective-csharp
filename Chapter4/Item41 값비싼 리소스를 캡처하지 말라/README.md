# 41\. 값비싼 리소스를 캡처하지 말라

## 41.1 클로저

자신이 선언된 스코프의 **지역변수를 캡처(capture)** 하여 그 스코프가 끝난 이후에도 사용할 수 있도록 **수명을 연장**한 함수(델리게이트)다.

즉, 함수(람다 / 익명 메서드)는 자기 자신만 전달되는 것이 아니라, **그 함수가 의존하는 바깥 환경(environment)까지 함께 묶여 하나의 실행 단위**로 만들어진다.

이 때문에 이러한 구조를 *닫힌 함수(closed function)*, 즉 **클로저**라고 부른다.

### 바깥 환경과 함께 묶인 구조

함수는 원래 **자기 코드만 있으면 실행 가능**하다.

```csharp
Func<int,int> square = x => x * x;
```

*   이 함수는 외부 변수에 의존하지 않는다.
*   입력값 `x`만 있으면 언제든 실행 가능하다.
*   따라서 **클로저가 생성되지 않는다.**

---

반면, 다음 함수는 다르다.

```csharp
int factor =10;

Func<int,int> multiply = x => x * factor;
```

*   `factor`는 함수 내부에 선언된 변수가 아니다.
    
*   이 함수는 `factor` 없이는 실행될 수 없다.
    
*   즉, 함수는 **외부 환경에 의존**하고 있다.
    
*   이 경우 컴파일러는 다음과 같이 처리한다.
    
    → “이 함수가 나중에 실행될 때도 factor가 필요하니, 함수 코드와 `factor`를 함께 묶어 하나의 객체로 만들자.”
    
    → 즉, 내부적으로 다음과 같은 구조가 생성된다.
    
    ```
    [ 함수 코드 ] +[ 함수가 참조하는 외부 변수들 ]
    ```
    
    이렇게 만들어진 객체가 바로 **클로저**다.
    

### 클로저가 만들어지는 상황

다음 요소가 **지역변수**를 사용하면 클로저가 만들어질 수 있다.

*   람다식 (`() => ...`)
*   익명 메서드 (`delegate { ... }`)
*   LINQ 쿼리
*   `yield return` (iterator)
*   비동기 메서드 (`async/await`)

단, **지역변수를 사용하지 않으면 클로저는 생성되지 않는다.**

### 컴파일러 관점에서 본 클로저

```csharp
int counter = 0;

Func<int> f = () => counter++;

Console.WriteLine(f()); // 0
Console.WriteLine(f()); // 1
```

컴파일러는 위의 코드를 대략 이런 형태로 바꾼다.

```csharp
class Closure
{
    public int counter;      // 지역 변수(외부 변수) -> 클래스의 필드

    public int Increment()   // 람다 -> 클래스의 인스턴스 메서드
    {
        return counter++;
    }
}

// 사용 코드
var c = new Closure();
c.counter = 0;

Func<int> f = new Func<int>(c.Increment);

```
| 원래 코드 개념 | 컴파일 후 |
| --- | --- |
| 지역변수 | 숨겨진 클래스의 필드 |
| 람다/익명 메서드 | 클래스의 인스턴스 메서드 |
| 캡처 | 인스턴스 필드 접근 |

**⇒ 이 “숨겨진 클래스 인스턴스”가 바로 클로저다**


## 41.2 클로저의 햄심 특성 : 수명 연장

### 일반적인 지역 변수

*   지역변수는 블록(메서드)이 끝나면 스택에서 사라진다.
*   그 변수가 가리키던 힙 객체도 더 이상 참조가 없으면 GC 대상이 된다.

### 클로저가 캡쳐한 변수

*   컴파일러가 숨은 클래스(closure class)를 만들고 **캡쳐된 지역 변수는 그 클래스의 필드**로 올라간다.
    
*   지역변수가 → 힙 객체의 필드로 승격되면서 델리게이트가 살아있는 한 절대 GC되지 않는다.
    
*   **델리게이트가 살아있는 한**
    
    **→ 그 델리게이트가 붙잡고 있는 closure 인스턴스도 살아 있고**
    
    → **closure 필드 (캡처된 변수)도 계속 살아있다.**
    

## 41.3 클로저로 인해 발생하는 문제

### 41.3.1 값비싼 리소스 참조 문제

클로저는 캡처된 변수의 수명을 **클로저 자체의 수명만큼 연장**한다.

캡처된 변수가 다음과 같은 **값비싼 리소스**라면 문제가 된다.

*   파일 스트림 (`StreamReader`, `FileStream`)
*   DB 커넥션
*   네트워크 소켓
*   대형 메모리 버퍼
*   `IDisposable`을 구현한 객체

이러한 리소스는 **명확한 시점에 해제되어야 한다.**

그러나 클로저에 의해 캡처되면, 해당 리소스의 수명이 클로저의 수명에 종속되어 해제 시점을 예측하기 어렵고, 예외 혹은 성능 저하로 이어질 수 있다.

### 41.3.2 지연 실행과 리소스 캡처 문제

LINQ와 iterator는 기본적으로 **지연 실행(lazy execution)** 모델을 따른다.

```csharp
var query = source.Where(x => x >10);
```

*   위 시점에서는 아무 작업도 수행되지 않고,
*   실제 실행은 `foreach`로 열거할 때 이루어진다.

이 특성과 클로저가 결합되면 다음과 같은 문제가 발생한다.

```csharp
using (var reader = new StreamReader(File.OpenRead("test.txt")))
{
    rows = ReadNumbersFromStream(reader);
}

foreach (var row in rows) { ... }

```

*   `reader`는 반환된 `IEnumerable`에 의해 **클로저로 캡처**된다.
*   `using` 블록이 종료되면서 `reader`는 Dispose된다.
*   실제 파일 접근은 열거 시점에 발생한다.
*   결과적으로 `ObjectDisposedException`이 발생한다.

### 41.3.4 의도치 않은 수명 연장

C# 컴파일러는 일반적으로 **메서드당 하나의 클로저 클래스만 생성한다**.

따라서 메서드 내부에 여러 람다식이나 쿼리 표현식이 존재하더라도, 이들이 캡처하는 지역변수들은 **하나의 클로저 객체에 함께 저장**된다.

이로 인해 개발자가 “이미 사용이 끝났다”고 생각한 객체가, 반환된 람다나 `IEnumerable`에 의해 **의도치 않게 수명이 연장**될 수 있다.

```csharp
private static IEnumerable<int> LeakingClosure()
{
    var filter = new ResourceHogFilter(); // 값비싼 리소스
    var source = new CheapNumberGenerator();
    var results = new CheapNumberGenerator();

    var importantStatistic =
        (from n in source.GetNumbers(50)
         where filter.PassesFilter(n) // filter를 캡처하는 람다
         select n).Average();

    return
        from n in results.GetNumbers(100)
        where n > importantStatistic // importantStatistic을 캡처하는 람다
        select n;
}
```

개발자 관점 ⇒ **`return` 시점에서 `filter`는 더 이상 필요가 없으니 gc의 대상이 될 것이라 생각한다.**

*   `filter`는 `importantStatistic` 계산에만 사용된다.
*   `Average()`를 호출했으므로 계산은 이미 끝났다고 생각한다.
*   반환되는 쿼리에는 `filter`가 직접 사용되지 않는다.

실제 컴파일러 동작 ⇒ `filter` 는 GC 대상이 아니다.

*   `importantStatistic`이 반환된 쿼리에서 사용되므로 클로저에 캡처된다.
    
*   같은 메서드에서 `filter` 역시 람다에 의해 캡처된다.
    
*   컴파일러는 **이 두 변수를 하나의 클로저 객체 필드로 묶는다**.
    
    ```csharp
    // 컴파일러가 내부적으로 만든 클래스
    
    private sealed class Closure
    {
        public ResourceHogFilter filter;
        public double importantStatistic;
    
        public bool Predicate(int n)
        {
            return n > importantStatistic;
        }
    }
    
    ```
    
    ```csharp
    // LeakingClosure()는 사실상 이렇게 된다.
    var c = new Closure();
    c.filter = new ResourceHogFilter();
    
    c.importantStatistic =
        source.GetNumbers(50)
              .Where(n => c.filter.PassesFilter(n))
              .Average();
    
    return results.GetNumbers(100)
                  .Where(n => c.Predicate(n));
    
    ```
    
*   `LeakingClousre()` 메서드에서 반환된 IEnumerable에서
    
    → LINQ의 Where는 내부적으로 델리게이트를 저장한다.
    
    → 이 델리게이트는 `c.Predicate` 라는 인스턴스 메서드를 참조한다.
    
    → Clousre 객체 c를 참조
    
    → Clousre는 filter를 참조하기 때문에 결과적으로 `filter`도 함께 살아남는다 → GC 대상이 아니다.
    

이렇기 때문에 의도치 않은 리소스 점유가 발생하게 되고 성능 저하가 발생할 수 있다.

### 41.3. 멀티스레드에서의 캡처

*   캡처된 변수는 클로저 객체의 **공유 필드**가 된다.
*   여러 스레드가 동시에 접근하면 **레이스 컨디션**이 발생할 수 있다.

## 41.4 해결 방법

클로저로 인한 리소스 수명 문제를 피하려면, **클로저의 수명과 값비싼 리소스의 수명을 분리**하는 방향으로 코드를 설계해야 한다.

### 41.4.1 메서드를 분리하여 클로저도 분리한다.

*   하나의 메서드에서 여러 람다를 사용하면, 컴파일러가 하나의 클로저 객체에 모든 캡처 변수를 묶을 수 있다.
*   값비싼 리소스를 사용하는 계산은 **별도의 메서드**로 분리하여 클로저가 분리되도록 한다.
*   반환되는 쿼리는 **값만 캡처**하도록 구성한다. (값비싼 리소스를 붙잡지 않도록 한다!)

```csharp
// LeakingClosure() 메서드 분리

ouble CalculateAverage()
{
    var filter = new ResourceHogFilter();
    var source = new CheapNumberGenerator();

    return (from n in source.GetNumbers(50)
            where filter.PassesFilter(n)
            select n).Average();
}

IEnumerable<int> SafeClosure()
{
    var avg = CalculateAverage(); // 값만 캡처 (객체를 캡처하는 것보다 가볍고 안전)
    var results = new CheapNumberGenerator();

    return results.GetNumbers(100)
                  .Where(n => n > avg);
}

```

### 41.4.2 즉시 실행으로 클로저 수명을 끊는다

*   지연 실행(`Where`, `Union`, `Select` 등)은 캡처된 객체의 수명을 **열거 시점까지 연장**시킨다.
*   필요한 계산이 끝났다면 다음과 같이 **즉시 실행 메서드**를 사용하여 앞 부분 계산을 즉시 종료시켜 값비싼 객체가 더 이상 클로저에 포함되지 않도록 한다.
    *   `Average()`
    *   `ToList()`
    *   `ToArray()`
    *   `Max()`, `Min()`

### 41.4.3 리소스를 캡처한 `IEnumerable`을 외부로 반환하지 않는다

파일, 스트림, DB 커넥션과 같은 리소스를 사용하는 경우:

*   리소스는 **함수 내부에서 열고 닫는다**
*   외부에는 `IEnumerable`을 반환하지 않는다
*   대신 **“무엇을 할지”를 델리게이트로 전달받는다**

→ 리소스의 생명주기를 함수 내부에서 완전히 통제할 수 있다.

```csharp
// 리소스를 캡처한 IEnumerable 반환 (X) 
IEnumerable<int> ReadNumbers(string path)
{
    var reader = new StreamReader(path); // Dispose되지 않아 리소스 누수 가능
    return reader.ReadLine()  // 실제 파일 읽기는 열거 시점에 발생하기 때문에 나중에 reader를 사용해야 함 (오랫동안 참조됨)
                 .Split(',')
                 .Select(int.Parse);
}

T ProcessFile<T>(string path, Func<IEnumerable<int>, T> action)
{
    using var reader = new StreamReader(path); // 리소스 생명주기가 이 함수 내부에서 완결됨

    var numbers =
        reader.ReadLine()
              .Split(',')
              .Select(int.Parse); // 지연실행이지만, using 범위 안에서 호출하므로 reader가 살아있는 동안 열거됨

    return action(numbers);
}
```

### 41.4.4 `using + yield return` 사용 시 재열거를 주의한다

*   `using`과 `yield return`을 함께 사용하는 패턴은 **올바른 사용**이다.

```csharp
public IEnumerable<int> SomeFunction()
{
    using var g = new Generator();
    while (true)
        yield return g.GetNextNumber();
}
```

*   단, **같은 시퀀스를 여러 번 열거하면**:
    *   매 열거마다 메서드가 다시 실행되고
    *   리소스도 매번 새로 생성·해제된다
*   파일이나 스트림을 재사용하는 설계라면 **한 번만 열거하도록 구조를 명확히 해야 한다.**

### 41.4.5 멀티스레드 환경에서 캡처된 가변 상태를 공유하지 않는다

클로저에 캡처된 변수는 **공유 상태(shared mutable state)** 가 되기 때문에 여러 스레드에서 접근 시 레이스 컨디션 발생 가능하다.

클로저가 **가변 상태를 캡처하지 않도록** 설계해야 한다.
