## 4. `string.Format()`을 보간 문자열로 대체하라

### 기존 방식: 문자열 포맷팅

* `string.Format()`

  ```csharp
  string format ="Name: {0}, Age: {1}, Country: {2}";
  string result =string.Format(format,"Bob",25,"USA");
  Console.WriteLine(result);
  
  ```

* 단점

  * 가독성이 떨어진다.

  * 실수해도 컴파일러가 잡아주지 않는다.

    * 포맷 인자 개수 불일치

    * `{0}`, `{1}` 순서 실수

  * 포맷 문자열과 인자 리스트가 분리된 구조

    → 항상 실행해서 결과가 정확한지 직접 확인해야 한다.

 

### 해결책: 문자열 보간 (C# 6.0+)

* 기본 사용법

  ```csharp
  string name ="Alice";
  int age =30;
  string city ="Wonderland";
  
  string result =$"Name: {name}, Age:{age}, City:{city}";
  
  
  ```

  * `$`로 문자열 보간 시작

  * `{}` 안에는 변수뿐 아니라 표현식(expression) 사용 가능

* 원리

  <figure><p>📌 C# 6.0 기준으로, C# 9.0 이후부터 박싱 문제가 해결됐다.</p></figure>

  * 실제로 C# 컴파일러가 `param`을 이용해, `object` 배열을 전달하는 기존 포맷팅 함수를 호출하도록 코드를 생성한다.

  * `object` 배열이기 때문에 표현식(변수)에 값 타입이 들어가면 박싱 발생할 수 있다. 🚨

    ```csharp
    Console.WriteLine($"The value of pi is {Math.Pi}"); // ❌ 박싱
    Console.WriteLine($"The value of pi is {Math.PI.ToString(“F2”))}”); // ✅
    
    ```

* 장점

  * 코드 가독성 향상

  * 컴파일러가 정적 타입 검사를 수행한다.\
    → 개발자의 실수 미연에 방지할 수 있다.

  * 기존 방식에 비해 문자열 생성 표현식이 풍성하다.

    * 조건 연산자(삼항 연산자), `null` 연산자 등과 결합하여 사용 가능

      <figure><p>❌ <code>while</code>, <code>if/else</code>문과 같이 제어 흐름을 변경하는 코드는 쓸 수 없다.</p></figure>

    * 중첩 표현식 사용 가능

      <figure><p>🚨 그러나 복잡한 코드는 오히려 가독성을 해친다.</p></figure>

* 주의할 점

  * 조건 표현식 `:`와 문자열 보간을 함께 사용하면 충돌이 발생할 수도 있다.

    ```csharp
    Console.WriteLine($”The value of pi is {round ? Math.PI.ToString() : Math.PI.ToString(“F2”)}”); // ❌ 
    Console.WriteLine($”The value of pi is {(round ? Math.PI.ToString() : Math.PI.ToString(“F2”))}”); // ✅
    
    ```

    * C#에서는 `:`를 포맷 문자열의 시작을 나타내는 것으로 판단하기 때문에, `()`를 사용해서 해결할 수 있다.

  * 문자열 보간의 결과는 하나의 문자열이다

    → SQL 명령을 만들거나, 객체/데이터로 재해석이 필요한 문자열을 생성하는 경우 주의해야 한다.

 

### C# 10 / .NET 6 이후

* 보간 문자열은 더 이상 `string.Format()`으로 변환되지 않고, 대신 `InterpolatedStringHandler`를 사용한다.

* 장점

  * **`InterpolatedStringHandler.AppendFormmated<T>(T value)`** 형태로 제너릭 기반이기 떄문에 박싱이 발생하지 않는다.

  * 또한, 내부적으로 `ToString()`을 호출하기 떄문에, 보간 문자열에서 이를 직접 호출할 필요가 없다.


</br>
<figure><p>👨‍🏫 <strong>그럼에도 불구하고 <code>string.Format()</code>을 사용해야 하는 경우</strong></p><ul><li><p>포맷 문자열이 기획 데이터 시트 등 외부 데이터에서 관리되는 경우,<br>문자열 보간을 사용할 수 없으므로 <code>string.Format()</code>을 사용할 수밖에 없다.</p><ul><li><p>예:&nbsp;"{0}이(가) {1}만큼 강화되었습니다."</p></li></ul></li><li><p>그 외의 대부분의 상황에서는 문자열 보간을 사용하는 것이 좋다.&nbsp;<br>(특히 코드 내부에서 로그를 남길 때 유용하다.)</p></li></ul></figure>
