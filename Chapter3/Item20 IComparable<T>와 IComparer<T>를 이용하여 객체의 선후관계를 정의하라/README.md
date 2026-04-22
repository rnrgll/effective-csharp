## 20. `IComparable<T>`와 `IComparer<T>`를 이용하여 객체의 선후 관계를 정의하라

### 객체의 선후 관계

* C#에서는 사용자 정의 타입의 선후 관계를 정의하기 위해, 다음 두 가지 표준 인터페이스를 제공한다.

  1. `IComparable<T>`: 타입의 기본 선후 관계 정의

  2. `IComparer<T>`: 타입의 추가적인 정렬 기준 정의

<figure><p>객체의 동일성 비교</p><ul><li><p>선후 관계와 동일성은 의미가 다르다.</p></li><li><p>보통 참조 타입에 대해서 선후 관계 비교는 객체의 내용에 대해서지만,<br><code>Eqauls</code>는 두 객체의 참조 자체를 비교한다.</p></li></ul></figure>

</br>

### `IComparable<T>`

* 타입의 기본 선후 관계 정의하는 데 사용한다.

  * 제너릭 버전 `IComaprable<T>`

  * 비제너릭 버전 `IComparable`

  <figure><p>📌 비제너릭 버전은 <code>object</code> 타입 기반으로 박싱/언박싱 문제가 발생하지만, 오래된 API와의 호환성을 위해 반드시 함께 구현해야 한다.</p></figure>

* 예시

  ```csharp
  // IComparable<T>와 IComparable 모두 구현
  public struct Customer : IComparable<Customer>, IComparable
  {
      private readonly string name;
  
      // IComparable<Customer> 멤버
      public int CompareTo(Customer other) => name.CompareTo(other.name);
  
      // IComparable 멤버
      int IComparable.CompareTo(object obj)
      {
          if (!(obj is Customer))
              throw new ArgumentException("Argument is not a Customer", "obj");
  
          Customer other = (Customer)obj;
          return this.CompareTo(other);
      }
  }
  ```

  * 추가적으로 관계 연산자를 오버로딩해 논리적인 일관성을 유지하는 것이 좋다.

    ```csharp
    public static bool operator <(Customer left, Customer right) => left.CompareTo(right) < 0;
    public static bool operator >(Customer left, Customer right) => left.CompareTo(right) > 0;
    public static bool operator <=(Customer left, Customer right) => left.CompareTo(right) <= 0;
    public static bool operator >=(Customer left, Customer right) => left.CompareTo(right) >= 0;
    ```

  * `CompareTo` 함수의 작동 방식
    * `this.CompareTo(other)` 호출 기준
      * `this < other` → 음수 반환
      * `this == other` → 0 반환
      * `this > other` → 양수 반환

 </br>

* 비제너릭 버전 `IComparable`을 사용할 때 주의사항

  * 비제너릭 버전은 성능 문제(박싱/언박싱)이 있기 때문에, 제너릭 버전를 기본으로 사용한다.

  * 명시적 인터페이스 구현(`IComparable.CompareTo`)를 통해 평소에는 비제너릭 버전에 대한 접근을 숨기고, `object` 타입 기반의 구형 API에 의해 호출될 때만 비제너릭 버전이 사용된다. 이 경우 명시적인 형변환이 필요하다.

  * 코드 예시

    ```csharp
    // 정의
    public struct Customer : IComparable<Customer>, IComparable
    {
    	int IComparable.CompareTo(object obj) 
    	{ 
    	}
    }
    
    // 사용
    Customer c1;
    Customer c2;
    if (((IComparable)c1).CompareTo(c2) > 0) 
    	Console.WriteLine("Customer one is greater");
    ```

</br>

### `IComparer<T>`

* 타입의 기본 선후 관계 외에도 추가적인 선후 관계(정렬 기준)를 외부에서 정의하는 데 사용된다.

  <figure><p>📌<code>IComparable&lt;T&gt;</code>와 마찬가지로, 하위 호환성을 위해 비제너릭 버전도 함께 구현해야 한다.</p></figure>

* 활용

  * 객체의 기본 선후 관계 외에도 다양한 정렬 기준을 정의하고 싶을 때

  * 소스 코드를 수정할 수 없는 외부 라이브러리 객체를 정렬하고자 할 때

  <figure><p>👨‍🏫 인벤토리 정렬, UI 정렬 등의 상황에서 자주 사용한다.<br>​​​​​​그런데 잘못 구현하기 쉬우니 주의해야 한다❗</p><ul><li><p>​​​​​​Comparer는 비교 결과가 반드시 일관성을 가져야 한다.&nbsp;<br>예를 들어&nbsp;<code>(A, B)</code> 비교 결과와 <code>(B, A)</code> 비교 결과는&nbsp;반드시 서로 반대가 되어야 한다.</p></li><li><p>이 규칙이 깨지면 정렬 과정에서 예외가 발생한다.</p></li><li><p>단순한 값 비교가 아니라 조건 기반 우선순위를 적용할 때, 동일한 조건을 가진 두 객체에 대한 처리를 잊어버려 문제가 발생할 수 있다.</p></li></ul><p>재현이 어렵고 실제 서비스 환경에서 장애로 이어질 수 있으므로,<br>Comparer 구현 시 일관성을 보장하기 위해 별도의 Comparer 디버거도 사용한다.&nbsp;&nbsp;</p></figure>

* 예시

  ```csharp
  // 고객을 기본 선후 관계 기준인 이름 대신에, 매출을 기반으로 정렬하고 싶을 때
  private class RevenueComparer : IComparer<Customer>
  {
  	int IComparer<Customer>.Compare(Customer left, Customer right) =>
      left.revenue.CompareTo(right.revenue);
  }
  
  private static Lazy<RevenueComparer> revComp =
  	new Lazy<RevenueComparer>(() => new RevenueComparer());
      
  public static IComparer<Customer> RevenueCompare => revComp.Value;
  ```

  <figure><p>정렬 기준은 보통 상태를 가지지 않기 때문에, 객체 생성 오버헤드를 줄이기 위해 <code>static</code> 인스턴스로 사용하는 것이 좋다.</p></figure>

</br>
<figure><p><strong><code>Comparison&lt;T&gt;</code> 델리게이트</strong></p><ul><li><p>정렬 기준이 일회성이거나, 복잡한 클래스 정의를 하고싶지 않을 때 사용하는 델리게이트</p></li><li><p>C#에 제너릭 기능이 도입된 이후, 대부분의 API는 해당 델리게이트를 이용한다.</p></li><li><p>예시</p><pre><code class="language-csharp">public static Comparison&lt;Customer&gt; CompareByRevenue =&gt;
	left, right) =&gt; left.revenue.CompareTo(right.revenue);</code></pre></li></ul></figure>

 
