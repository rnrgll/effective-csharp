## 2. `const`보다는 `readonly`가 좋다

### 컴파일 타임 상수: `const`

* 컴파일 타임에 값이 결정되는 상수

* IL 코드로 컴파일될 때 변수가 리터럴로 치환된다. (참조가 아닌 복사!)

  ```csharp
  if (myDataTime.Year == Millenium) // C#
  if (myDataTime.Year == 2000) // IL
  
  ```

* 적용 가능한 타입

  * C#에 내장된 숫자형 / `enum` / 문자열 / `null`

  * 이유: 내장 자료형인 경우에만 컴파일 타임에 상수를 리터럴로 대체할 수 있음

  <figure><p>🚨 그 외에는 컴파일 오류가 발생한다.</p></figure>

  ```csharp
  private const DataTime time = new DataTime(2000, 1, 1, 0, 0, 0); // ❌
  private readonly DataTime time = new DataTime(2000, 1, 1, 0, 0, 0); // ✅
  
  ```

* 선언 및 초기화

  * 클래스의 멤버 필드 & 메서드 내부에서 지역 변수로 선언할 수 있다.

  * 반드시 선언과 동시에 초기화해야 한다.

* `const`는 `static`의 의미를 포함한다.

  * → 특정 객체(인스턴스)에 속한 것이 아니라, 클래스 자체에 속한 값이다.

 

### 런타임 타입 상수: `readonly`

* 런타임에 값이 결정되는 상수

* 컴파일 타임에 값으로 치환되지 않고, 상수 변수에 대한 참조로 유지된다.

* 적용 가능한 타입

  * 내장 자료형뿐만 아니라, 어떤 타입과도 사용할 수 있다.

* 선언 및 초기화

  * 클래스의 필드로만 선언할 수 있다. (메서드 내에서 선언할 수 없다! ❌)

  * 멤버 초기화 구문뿐만 아니라 생성자를 통해 초기화할 수 있다.

* 클래스 내에서 런타임 상수를 정의하는 경우, 인스턴스별로 다른 값을 가질 수 있다.

  * 모든 인스턴스가 동일한 값을 가지길 원할 경우, `static` 키워드를 붙여야 한다.

    ```csharp
    public static readonly int Value;
    
    ```

  * 이때, 정적 생성자에서만 초기화할 수 있다.

 

### 런타임 타입 상수가 더 좋은 이유

> 💡 전체적으로 더 유연하고 유지보수성이 좋기 때문에, 예외적인 경우를 제외하는 런타임 타입 상수를 사용하는 것이 좋다!

* 사용할 수 있는 타입의 범위가 훨씬 넓다

* 초기화 방식이 더 유연하다. → 실행 환경에 따라 값이 달라지는 상수를 만들 수 있다.

* 어셈블리 경계를 넘을 때 유지보수성이 훨씬 좋다.

  1. `Insfrastructure` 어셈블리에 다음 코드가 있다고 가정한다.

     * 런타임 상수

       ```csharp
       public class UsefulValues
       {
           public static readonly int StartValue = 5;
           public static readonly int EndValue = 7;
       }
       
       ```

     * 컴파일 타임 상수

       ```csharp
       public class UsefulValues
       {
           public const int StartValue = 5;
           public const int EndValue = 7;
       }
       
       ```

  2. 다른 어셈블리에서 상수를 아래와 같은 형태로 사용한다고 가정한다.

     ```csharp
     for (int i = UsefulValues.StartValue; i < UsefulValues.EndValue; i++)
         Console.WriteLine("value is {0}", i);
     
     ```

  3. 이 상태에서 `StartValue = 10`, `EndValue = 12`로 수정했다고 가정하자.

  4. 결과 : 컴파일 타임 상수를 사용한 경우, 다른 어셈블리의 변경 사항이 제대로 반영되지 않음

     <table><tbody><tr><th><p>방식</p></th><th><p>필요한 재빌드</p></th></tr><tr><td><p><code>static readonly</code></p></td><td><p><code>UsefulValues</code>가 있는 어셈블리만 재빌드</p></td></tr><tr><td><p><code>const</code></p></td><td><p><code>UsefulValues</code> + 이를 참조하는 모든 어셈블리 재빌드 필요</p></td></tr></tbody></table>

 

### 컴파일 타임 상수를 사용하는 경우 (예외적)

* 컴파일 타임 상수가 문법적으로 반드시 필요한 경우 (컴파일시 사용되는 상숫값을 정의할 때)

  * 특성(Attribute) 인자

  * `switch`문의 `case` 레이블

  * `enum` 정의

* 값이 절대로 변하지 않고, 성능이 극단적으로 중요한 경우

  <figure><p>💬 극단적인 경우가 아니라면 실제로 성능 개선 효과는 크지 않다.</p></figure>

 
