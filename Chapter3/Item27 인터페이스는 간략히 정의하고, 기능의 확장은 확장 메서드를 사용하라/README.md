## 27. 인터페이스는 간략히 정의하고, 기능의 확장은 확장 메서드를 사용하라

> 인터페이스에는 반드시 필요한 기능만을 포함하도록 간단하게 정의하고, 사용자 편의를 위해 다양하게 제공하려는 기능은 확장 메서드 방식으로 작성해야 한다.


### 인터페이스 대신 확장 메서드를 사용해야 하는 이유

#### 확장 메서드 (Extension Method)
- 기존 클래스나 구조체의 정의를 변경하지 않고, 외부에서 새로운 메서드를 추가한 것처럼 사용할 수 있게 하는 C#의 기능
- 사용 목적

   * 코드 수정이 불가능한 타입 확장\
     **:** .NET 기본 타입(`string`, `int`, `List<>`)이나 외부 라이브러리에 기능을 추가할 때 사용

   * 유틸/헬퍼 메서드의 표현력 및 가독성 개선

     * 기존: `Helper.WordCount(text);`

     * 확장 메서드: `text.WordCount();`

- 구현 규칙
  1. `static` 클래스 안에 정의해야 한다.
  
  2. 메서드는 `static` 이어야 한다.
  
  3. 첫 번째 매개변수 앞에 `this` 키워드를 사용해 확장 대상 타입을 지정한다.

- 예시
  ```csharp
  // 문자열 반전을 위한 string 클래스의 확장 메서드 
  
  // ==============================================
  // 정의
  public static class StringExtension
  {
      public static string ToReverse(this string str)
      {
          char[] charArray = str.ToCharArray();
          Array.Reverse(charArray);
          return new string(charArray);
      }
  }
  
  // ==============================================
  // 사용
  string myName = "ABC";
  string reversed = myName.ToReverse(); // 마치 원래 클래스에 존재하는 메서드처럼 사용
  Console.WriteLine(reversed);
  ```
    
#### 확장메서드를 사용하는 이유

* 확장 메서드를 통해 풍부한 많은 기능을 멤버 메서드처럼 사용할 수 있다.

* 인터페이스에는 반드시 필요한 최소한의 기능만 정의하면 된다.\
  → 인터페이스를 구현하는 클래스에서 필수로 작성해야 하는 메서드 수를 줄일 수 있다.

* 기존 코드를 수정하지 않고도 새로운 기능을 추가할 수 있다.

<figure><p>👨‍🏫 <strong>확장 메서드를 사용하면 좋은 경우</strong>​​​​</p><ul><li><p>범용 기능을 추가하고 싶을 때</p><ul><li><p>기존 타입에 있을 것 같지만 실제로 없는 기능을 공통적으로 사용하고 싶을 때 사용하면 좋다.</p></li></ul></li><li><p>사용 범위를 의도적으로 제한하고 싶을 때</p><ul><li><p>특정 피처 내부에서는 자주 쓰이지만, 전체 시스템에 노출시키고 싶지 않은 기능이 있을 수 있다.</p></li><li><p>이 경우 클래스 내부나 범용 유틸리티로 두는 대신, 해당 피처 네임스페이스에 확장 메서드로 정의해 사용 범위를 자연스럽게 제한할 수 있다.</p></li><li><p>특정 피처 전용 기능임을 드러내기 위해, 메서드 이름을 명확하고 구체적으로 짓는 것이 중요하다.</p></li></ul></li></ul></figure>


</br>

### 예시

* `System.Linq.Enumerable` 클래스

  * 인터페이스 (`IEnumerable<T>`) : **`GetEnumerator()`** 하나의 메서드 하나만 정의되어 있다.

  * 확장 메서드 (`System.Linq.Enumerable`)\
    : `Where`, `OrderBy`, `ThenBy`, `GroupBy`, `Select` 등 50개 이상의 메서드가 확장 메서드로 구현되어 있다.

  * 사용자가 커스텀 클래스를 만들 때, `GetEnumerator` 메서드 하나만 구현하더라도, 다양한 LINQ 기능을 사용할 수 있다.

* `Comparable` 클래스

  ```csharp
  public static class Comparable
  {
      public static bool LessThan<T>(this T left, T right) where T : IComparable<T> 
          => left.CompareTo(right) < 0;
      public static bool GreaterThan<T>(this T left, T right) where T : IComparable<T> 
  		=> left.CompareTo(right) < 0;
      public static bool LessThanEqual<T>(this T left, T right) where T : IComparable<T> 
  		=> left.CompareTo(right) <= 0;
      public static bool GreaterThanEqual<T>(this T left, T right) where T : IComparable<T> 
  		=> left.CompareTo(right) <= 0;
  }
  ```

   기존의 메서드보다 가독성이 좋고 직관적인 메서드를 사용자에게 제공할 수 있다.

</br>

### 확장 메서드 사용시 주의할 점

* 문제

  : 인터페이스를 구현한 클래스가 확장 메서드와 동일한 메서드를 구현하고 있는 경우, 메서드 호출 순서에 따라 충돌이 발생할 수 있다.

  <figure><p><strong>컴파일러의 메서드 호출 규칙</strong></p><ul><li><p>클래스 타입으로 호출 시<br>: 클래스 내부에 정의된 메서드가 우선 호출된다. (확장 메서드 무시)</p></li><li><p>인터페이스 타입으로 호출 시<br>: 만약 인터페이스 정의에 해당 메서드가 없다면, 확장 메서드가 호출된다.</p></li></ul></figure>

* 해결

  * 클래스 내부 메서드와 확장 메서드가 겹치지 않도록 인터페이스를 간결하게 유지한다.

  * 만약 어쩔 수 없이 겹친다면, 두 메서드가 의미적으로 완전히 동일한 동작을 하도록 구현해야 혼란을 막을 수 있다.

 

 
