# 3장. 제너릭 활용
## 제너릭 관련 기본 개념
### C# 코드가 실행되는 과정 (JIT 기준)
1. C# 컴파일 단계

   * C# 코드 → C# 컴파일러 → IL + 메타데이터 → `.dll/.exe`

2. 로드 단계

   * CLR이 로드된 `.dll/.exe`(IL)과 내부의 타입 정보를 읽는다.

3. JIT 컴파일 단계

   * 메서드가 처음으로 실제 호출될 때 JIT가 메서드 단위로 IL을 기계어로 변환한다.

   * 🚨 어셈블리가 로드되자마자 모든 클래스의 머신 코드를 생성하는 게 아니다.

4. 실행 단계

   * CPU가 기계어를 읽고 실행한다.

   * 스택/힙 할당, 메서드 호출, 예외 처리, 스레드 관리 → CLR이 담당



### 제너릭(Generic)

* 데이터 타입을 확정하지 않고 코드를 작성한 뒤, 사용할 때 타입을 지정하는 기법

  ```csharp
  public class MyList<T> // 제너릭 클래스
  {
      public void Add(T item) { ... } // 제너릭 메서드
  }
  ```

* 책 용어 정리

  * 제너릭 타입 정의

    * 어떤 타입을 제너릭의 형태로 정의하는 것

    <figure><p>C#에서 제너릭을 적용할 수 있는 타입</p><p>: 클래스 / 메서드 / 컬렉션 / 구조체 / 인터페이스 / 델리게이트</p></figure>

  * 타입 매개변수 `T`

    * 제너릭 타입에서 특정 타입에 대한 형식 매개변수 (자리 표시자 역할)

    <figure><p>📌 제너릭 타입 내에 포함된 코드는 타입 매개변수로 어떤 타입이 지정되더라도 유효한 코드여야 한다.</p></figure>

  * 닫힌 제너릭 타입

    * 타입 매개변수(`T`)에 구체적인 타입을 지정한 경우

    * 컴파일러나 JIT가 실제로 객체를 만들 수 있는 상태

  * 열린 제너릭 타입

    * 타입 매개변수 중 하나 이상이 아직 지정되지 않은 상태의 제너릭 타입



### 제너릭 코드가 생성되는 과정

> 제너릭 타입의 타입 매개변수로 전달되는 타입의 종류(값 타입 or 참조 타입)에 따라 코드의 생성 과정이 다르다.

1. C# 컴파일 단계

   * IL에서 제너릭은 타입을 부분적으로 정의한 형태이다.

   * 제너릭 타입으로 인스턴스를 생성할 때 반드시 필요한 타입 매개변수에 대해서는 그 위치만을 표시해둔다.

2. JIT 컴파일 단계

   * JIT 컴파일러는 제너릭 클래스를 만나게 되면, 타입 매개변수를 확인하여 해당 타입에 가장 적합한 머신 코드를 생성하기 위해 노력한다.

     1. 닫힌 제너릭 타입을 표현하기 위한 런타임 정보(MethodTable)을 생성한다.

     2. 제너릭 메서드가 처음으로 호출될 때 네이티브 코드를 생성한다.

     <figure><p>모든 닫힌 제너릭 타입을 미리 네이티브 코드로 만들면 코드의 크기가 너무 증가하고, 전혀 만들지 않으면 실행 시 성능이 저하되기 때문에, JIT 시점에 필요한 것만 생성한다.</p></figure>

   * 타입 매개변수로 값 타입이 전달될 때

     * 각 값 타입마다 서로 다른 네이티브 코드(기계어)가 생성된다.

     * 이유 : 값 타입의 데이터 크기가 다르기 때문에

       ```csharp
       List<int>  // int: 4바이트
       List<long> // long: 8바이트
       List<bool> // bool: 1바이트
       // 모두 서로 다른 기계어 코드 가짐
       ```

   * 타입 매개변수로 참조 타입이 전달될 때

     * 단 하나의 네이티브 코드가 생성되고, 모든 참조 타입은 런타임에 동일한 머신 코드를 공유하게 된다.

     * 이유\
       : 참조(포인터)는 모두 동일한 크기(8바이트)이기 때문에, 메모리의 풋프린트에 영향을 주지 않는다.

       ```csharp
       List<string>, List<Stream>, List<object>, List<MyClass>
       // 모두 동일한 머신 코드 공유함
       ```

        

### 제너릭 장단점

* 코드의 크기

  * 일반적으로는 코드의 크기가 작아지지만 항상 그런 것은 아니다. (타입 매개변수를 어떻게 지정하느냐 / 얼마나 많은 수의 닫힌 제너릭 타입을 생성하느냐에 따라 다르다.)

  * 값 타입을 많이 사용하면 코드가 증가할 수 있다. (참조 타입은 코드 공유)

* 값 타입에서 발생하는 박싱/언박싱을 제거할 수 있다.

  <figure><p>코드의 크기 증가로 인한 단점보다, 이로 인한 성능 이점이 훨씬 크다!</p></figure>

* 타입 안정성을 제공한다.

  * 컴파일 타임에 타입 오류를 검출해, 런타임 오류를 사전에 방지한다.

  * 예시

    ```csharp
    // 제네릭 ❌
    ArrayList list = new ArrayList();
    
    list.Add(10);       // 정수 
    list.Add("Hello");  // 문자열
    
    int sum = 0;
    foreach (object obj in list)
    {
        sum += (int)obj; // string을 int로 형변환 -> 런타임 에러 🚨
    }
    
    
    // 제네릭 ✅
    List<int> list = new List<int>();
    
    list.Add(10);
    list.Add("Hello");   // 컴파일 에러 🚨
    
    foreach (int num in list)
    {
        sum += num;      // 별도의 타입 검사나 형변환(Casting)이 필요 없음
    }
    ```
</br>

> #### AOT 기준 제너릭
> - AOT 환경에서는 런타임에 제너릭 코드를 생성할 수 없기 때문에, 모든 제너릭 인스턴스는 컴파일 타임(빌드 타임)에 미리 생성되어야 한다.
> - AOT 컴파일러는 컴파일 타임에 다음을 수행한다.
>   1. 전체 코드를 분석하여 사용되는 닫힌 제너릭 타입 목록을 수집한다.
>   2. 해당 조합에 대한 네이티브 코드 생성 후 실행 파일에 포함한다.
> - 주의할 점 : 리플렉션 등으로 소스에 없는 제너릭 조합을 요청할 경우 실패할 수 있다. (특히 값 타입!)

</br>


> #### C++ 템플릿
> - 컴파일 타임에 타입 또는 값에 따라 코드를 생성하는 프로그래밍 기법
> - 타입 안전성보다 표현력과 성능을 중요시한다. : 특수화, 부분 특수화, 비타입 인자(값) 등 지원
> - 사용 방법 </br>
>   `template <typenmae T>`
> - 비교
>  <table><tbody><tr><th><p><strong>특성</strong></p></th><th><p><strong>C++ 템플릿 (Templates)</strong></p></th><th><p><strong>C# 제네릭 (Generics)</strong></p></th></tr><tr><td><p><strong>생성 시점&nbsp; &nbsp; &nbsp;</strong></p></td><td><p>컴파일 타임 (Compile-time)&nbsp; &nbsp; &nbsp;</p></td><td><p>런타임 / JIT&nbsp; &nbsp; &nbsp;</p></td></tr><tr><td><p><strong>코드 공유</strong></p></td><td><p>타입마다 별도 코드 생성<br>→ 코드 크기 증가</p></td><td><p>참조 타입은 코드 공유,<br>값 타입만 개별 생성</p></td></tr><tr><td><p><strong>유연성</strong></p></td><td><p>매우 높음</p></td><td><p>제한적 (타입 안전성 중시)</p></td></tr><tr><td><p><strong>제약 조건</strong></p></td><td><p>암시적 (컴파일 오류로 검증)</p></td><td><p>명시적 (<code>where</code> 절 필수)&nbsp; &nbsp; &nbsp;&nbsp;</p></td></tr></tbody></table>


</br>
</br>


## 아이템 18. 반드시 필요한 제약 조건만 설정하라

### 타입 매개변수에 대한 제약 조건

* 제네릭 타입 매개변수(`T`)로 전달할 수 있는 타입의 유형을 제한하는 방법

* `where` 키워드를 사용하여 정의한다.

* 제약 조건의 종류

  <table><tbody><tr><th><p><strong>제약 조건</strong></p></th><th><p><strong>의미</strong></p></th></tr><tr><td><p><code>where T : struct</code></p></td><td><p>값 타입만 가능 (Nullable 제외)</p></td></tr><tr><td><p><code>where T : class</code></p></td><td><p>참조 타입만 가능</p></td></tr><tr><td><p><code>where T : new()</code></p></td><td><p>매개변수가 없는 public 생성자 필수</p></td></tr><tr><td><p><code>where T : BaseClass</code></p></td><td><p>특정 클래스를 상속받아야 함</p></td></tr><tr><td><p><code>where T : Interface</code></p></td><td><p>특정 인터페이스를 구현해야 함</p></td></tr></tbody></table>

<figure><p>👨‍🏫&nbsp;만약 <code>enum</code>을 제약 조건으로 걸고 싶다면?</p><ul><li><p>이전에는 별다른 방법이 없어서, <code>where T : struct</code>을 이용해 값 타입 제약 조건을 거는 것이 최선이었다.</p></li><li><p>C# 7.3 (2018년) 이후부터는 <code>where T : System.Enum</code>을 사용할 수 있다.</p></li></ul></figure>

 

### 사용하는 이유 (장점)

* 컴파일러 & 다른 개발자에게 타입 매개변수 `T`에 대한 추가 정보를 줄 수 있다.

  <figure><p>👨‍🏫 특히 많은 사람들과 함께 개발하는 프로젝트인 경우,<br>다른 사람에게 타입에 대한 정보를 알리기 위한 목적이 크다.</p></figure>

* 타입 관련 에러를 컴파일 타임에 확인할 수 있다

* 런타임 형변환(`is`, `as`), 리플렉션, 예외 처리 등을 훨씬 줄일 수 있다.\
  → 코드 단순화 및 성능 향상


### 비교

* 제약 조건이 없는 경우

  * 컴파일러는 `T`를 `System.Object`로만 취급한다.\
    → `Object`가 제공하는 기본 기능 외에는 사용할 수 없다.

  * 예시 ❌

    ```csharp
    public static bool AreEqual<T>(T left, T right)
    {
        if (left == null) 
    		return right == null;
    
        if (left is IComparable<T>)
        {
            IComparable<T> lval = left as IComparable<T>;
            if (right is IComparable<T>)
                return lval.CompareTo(right) == 0;
            else
                throw new ArgumentException("Type does not implement IComparable<T>", 
    		        nameof(right));
        }
        else
        {
            throw new ArgumentException("Type does not implement IComparable<T>", 
    		    nameof(left));
        }
    }
    ```

    * 코드가 길고 복잡하며, 런타임 에러(`ArgumentException`)가 발생할 수 있다.

* 제약 조건을 설정한 경우

  * 컴파일러는 `T`가 제약 조건에 해당하는 기능을 가지고 있다고 가정하고 메서드 호출을 허용한다.

  * 예시 ✅

    ```csharp
    public static bool AreEqual<T>(T left, T right) 
        where T : IComparable<T> =>
    		return left.CompareTo(right) == 0;
    ```

    * `T`가 반드시 `IComparable<T>`를 구현해야 함을 컴파일러가 보장하므로, 바로 `CompareTo` 메서드를 호출할 수 있다.

    * 코드도 훨씬 간결하다! (런타임 형변환, 예외 처리 등등 모두 없음)



### 주의 사항

* 제너릭의 본질은 다양한 상황에서 적용할 수 있는 범용 타입이다.

  그러나, 제약 조건이 너무 많아지면 이를 만족시키기 위해 과도한 추가 작업을 수행해야 하고, 사용성이 떨어진다.

  **→ 제약 조건은 반드시 필요한 경우에만, 최소로 설정해야 한다.**

  <figure><p>일단은 제약 조건 설정해보고, 너무 많은 제약이 생겨 사용성이 떨어진다면, 제약 조건을 제거한 더 낮은 수준의 메서드를 사용하도록 하는 것이 좋다.</p></figure>

* 예시: 기본 생성자 제약 조건
  * 기본 생성자(`new`) 제약 조건
    * 제너릭 메서드 안에서 `new`를 통해 새로운 객체를 생성하고 싶을 때 사용한다. (해당 제약 조건이 없는데 `new`를 통해 객체를 생성하려고 하면, 컴파일 오류 발생)
    * 제너릭 타입이 “매개변수가 없는 public 생성자를 반드시 가져야 한다.”는 의미
  * `default(T)`
    * 어떤 타입(`T`)의 기본값을 반환하는 키워드
    * 값 타입일 경우 0, 참조 타입일 경우 null을 반환한다.
  * `new` 제약 조건은 매개변수가 없는 기본 생성자의 정의를 강제하기 때문에, 새로운 객체 생성이 꼭 필요한 게 아니라면 `default` 키워드를 통해 기본값만 반환하는 게 좋다.
