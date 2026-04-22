## 1. 지역변수를 선언할 때는 `var`를 사용하는 것이 낫다

### `var` 키워드의 의미

* 지역 변수의 타입을 암시적으로 선언할 수 있게 하는 키워드

  <figure><p>📌 지역 변수에만 사용 가능하다.</p></figure>

* 컴파일러는 컴파일 타임에 할당문 오른쪽 내용을 기반으로 타입을 추론(결정)한다.

  → 동적 타이핑이 수행되는 것이 아니다.

* 배경

  * C#의 익명 타입(anonymous type)을 지원하기 위해 필요해졌다.

  * 

  <figure><p>📝 익명 타입</p><ul><li><p>일반적으로 어떤 클래스를 사용하기 위해서는 클래스를 정의한 후 사용해야 한다.</p></li></ul><ul><li><p>C# 3.0부터는 클래스를 미리 정의하지 않아도 사용할 수 있는 익명 타입을 제공하는데, 이를 데이터 변수에 담기 위해 <code>var</code> 키워드를 사용한다.</p></li><li><p>(실제로는&nbsp;<code>&lt;&gt;f__AnonymousType0'2</code>와 같은 복잡한 이름을 가진다.)</p></li></ul></figure>

  1. 예시

     ```csharp
     var person = new {Name = "Kim, Age = 30};
     
     ```

### 장점

* 적절하게 사용하면 코드의 가독성을 높일 수 있다.

  → 개발자가 지엽적인 부분인 변수의 타입보다는, 변수의 의미 파악에 더 집중할 수 있다.

  <figure><p>📌 단, 변수의 이름이 변수의 역할을 명확하게 드러낼 수 있어야 한다!</p></figure>

  * 예시

    ```csharp
    Dictionary<int, Queue<string>> dict = new Dictionary<int, Queue<string>>(); // ❌
    var jobsQueuedByRegion = new Dictionary<int, Queue<string>>(); // ✅
    
    ```

* 올바르지 않은 타입을 명시적으로 지정하는 경우 발생하는 잘못된 동작을 방지할 수 있다.

  * 예: `IEnumerable<T>`와 `IQueryable<T>`

    ```csharp
    IEnumerable<T> data = query; // 실제로는 IQueryable<T>일 수도 있음 -> 성능 저하
    var data = query;            // 실제 반환 타입 유지
    
    ```

    <figure><p>📝 ‘아이템 42.&nbsp;<code>IEnumerable&lt;T&gt;</code> 데이터 소스와 <code>IQueryable&lt;T&gt;</code> 데이터 소스를 구분하라’에서 자세히 다룰 예정</p></figure>

     

### 추천

* 지역변수를 초기화하는 경우

* 변수의 타입이 명확한 경우

* 예시

  ```csharp
  var foo = new MyType(); // 생성자
  var thing = AccountFactory.CreateSavingAccount(); // 팩토리 메서드
  
  ```

 

### 주의

* 특정 메서드의 반환값을 저장할 변수를 `var`을 이용해 선언한 경우, 메서드/변수명을 명확하게 정의하지 않은 경우, 가독성이 떨어질 수 있다.

  ```csharp
  var result = someObject.DoSomeWork(anotherParameter);
  
  ```

* C#의 내장된 숫자 타입들을 사용할 때는 `var`을 사용하지 않는 것이 좋다.

  * C#의 내장 숫자 타입들은 매우 다양한 형변환 기능을 가지고 있고, 정밀도도 다르다.

  * 컴파일러는 계산식에 사용된 리터럴 상수들을 변수와 동일한 타입으로 변환해서 계산한다.\
    → 결과값에 차이가 생길 수 있다.

    ```csharp
    var total = 100 * f / 6;
    // f의 타입에 따라 total의 타입과 값이 달라진다.
    
    ```

* `var`는 컴파일 타임에 객체의 정적 타입만 추론할 수 있다.

  ```csharp
  Base b = new Derived(); // b의 타입: Base
  var d = b; // d의 타입: Base
  
  ```

 

<details><summary>👨‍🏫<strong> C++ <code>auto</code></strong></summary><ul><li><p>비교</p><table><tbody><tr><th><p><strong>특징</strong></p></th><th><p><strong>C++ auto</strong></p></th><th><p><strong>C# var</strong></p></th></tr><tr><td><p>타입 결정 시점&nbsp;</p></td><td><p>컴파일 타임 (정적 타이핑)</p></td><td><p>컴파일 타임 (정적 타이핑)</p></td></tr><tr><td><p>사용 위치</p></td><td><p>지역 변수, 리턴 타입, 매개변수</p></td><td><p>지역 변수만 가능</p></td></tr><tr><td><p>추론 형식</p></td><td><p>기본적으로 값 복사<br>(<code>auto&amp;</code>, <code>const auto</code> 등 명시 필요)</p></td><td><p>타입 자체에 포함된 속성을 따름</p></td></tr><tr><td><p>주요 용도</p></td><td><p>복잡한 템플릿 타입 축약, 제네릭 프로그래밍&nbsp;&nbsp;</p></td><td><p>긴 타입명 축약, 익명 타입(LINQ) 사용 시 필수&nbsp;&nbsp;</p></td></tr></tbody></table></li><li><p>C++에서 무분별한 <code>auto</code> 사용을 지양해야 하는 이유</p><ul><li><p>C++의 타입 시스템은 C#에 비해 복잡하다.<br>(참조, <code>const</code>, 값 범주, 포인터, 템플릿 등)</p></li><li><p><code>auto</code>를 사용하면 컴파일러가 타입을 기계적으로 추론한다.<br>기본적으로 값 복사가 선택되어 <code>const</code>, <code>&amp;</code>가 의도와 다르게 사라질 수도 있고, 그 외에도 본래의 의도와 다른 타입으로 추론될 수 있다.</p><p>→ C++에서 타입의 특성은 객체의 수명 / 소유권 / 복사 비용에 직접적인 영향을 주기 때문에 더 위험하다.</p></li><li><p>사용 범위가 <code>var</code>에 비해 훨씬 넓다.<br>→ 오히려 의도 표현이 약해지고 가독성이 저하될 수 있다.</p></li></ul></li></ul></details>

 

### 
