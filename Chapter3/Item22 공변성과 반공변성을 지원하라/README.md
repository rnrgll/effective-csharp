## 22. 공변성과 반공변성을 지원하라

### 타입의 가변성(variance)

* 타입 매개변수 간의 변환 가능성이 제네릭 타입에도 반영되는 성질

  <figure><p>↔️ 타입의 불변성 (invariant)</p></figure>

  * 공변성 (Convariance)

    * `X`를 `Y`로 바꾸어 사용 가능 → `C<X>`를 `C<Y>`로도 바꿔 사용 가능

      ⇒ `C<T>`는 공변

    * 더 구체적인 타입의 객체를 더 일반적인 타입의 변수에 할당할 수 있다.

  * 반공변성 (Contravariance)

    * `Y`를 `X`로 바꾸어 사용 가능 → `C<X>`를 `C<Y>`로도 바꿔 사용 가능

      ⇒ `C<T>`는 반공변

    * 더 일반적인 타입의 객체를 더 구체적인 타입의 변수에 할당할 수 있다.

* C#에서 가변성의 적용 범위

  * 참조 타입에만 적용된다.

  * 제너릭 인터페이스와 델리게이트 형식에 가변성을 적용할 수 있다.

 

### `in` / `out` 한정자

* C# 4.0 이후, 가변성 지원을 위해 도입된 한정자/키워드

* 제너릭 인터페이스나 델리게이트의 정의 부분에 해당 키워드를 추가해서 가변성을 지원할 수 있다.

  <figure><p>📌 타입 매개변수에 붙은 키워드 <code>in/out</code>과,<br>메서드 파라미터에 사용되는 키워드인 <code>in/out</code>은 서로 다른 의미이다.</p></figure>

* `out T`

  * `T`를 반환 위치에서만 사용하겠다고 선언해서, 공변성을 지원할 수 있게 하는 키워드

    <figure><p>반환 위치</p><ul><li><p>메서드의 리턴 타입, 프로퍼티의 <code>get</code> 접근자, 델리게이트의 일부 위치</p></li><li><p>이때 <code>T</code>는 읽기 전용으로만 사용할 수 있다.</p></li></ul></figure>

* `in T`

  * `T`를 입력 위치에서만 사용하겠다고 선언해서, 반공변성을 지원할 수 있게 하는 키워드

    <figure><p>입력 위치</p><ul><li><p>메서드의 파라미터, 프로퍼티의 <code>set</code> 접근자 등</p></li></ul></figure>

     

### 예시 코드로 보는 설명

* 전제 조건

  * X (자식): `Dog`

  * Y (부모): `Animal`

  * 관계: `Dog` → `Animal` (할당 가능)

* 불변성 (Invariance)

  * `List<T>`

  * 변환 불가 (`List<Dog>` ≠ `List<Animal>`)

  * 이유\
    : 읽기/쓰기가 다 되므로, 타입을 바꾸면 데이터 오염이 발생할 수 있다. 

    ```csharp
    List<Dog> dogs = new List<Dog>();
    
    // ❌ 컴파일 에러
    List<Animal> animals = dogs; 
    ```

  * ❓만약 공변성이 허용된다면?
     ```csharp
     animals.Add(new Cat()); // 동물 리스트니까 고양이 추가 가능
     Dog d = dogs\[0\]; // dogs 리스트에 고양이가 들어있음 -> 에러!🚨
     ```
     
* 공변성 (Convariance)

  * `IEnumerable<out T>`

  * 정방향 변환 가능 (`IEnumerable<Dog>` → `IEnumerable<Animal>`) 

    ```csharp
    IEnumerable<Dog> dogs = new IEnumerable<Dog>();
    
    // ✅ 공변성 적용 (Dog -> Animal)
    IEnumerable<Animal> animals = dogs; 
    foreach (Animal a in animals) 
    {
    }
    ```


  * 안전한 이유\
    : `animals`는 '읽기 전용' 인터페이스 → `Dog` 목록에 `Cat`을 추가할 위험이 없음
 
    * 👨‍🏫 또 다른 공변성이 적용된 예시: ​​`IReadOnlyList`

* 반공병성 (Contravariance)

  * `IComparable<in T>`

  * 역방향 변환 가능 (`IComparable<Animal>` → `IComparable<Dog>`) 

    ```csharp
    // 모든 동물을 특정 기준으로(예: 나이, 크기) 비교하는 범용 비교기
    class AnimalComparer : IComparable<Animal>
    {
        public int CompareTo(Animal other);
    }
    
    IComparable<Animal> aniComp = new AnimalComparer();
    
    //  ✅ 반공변성 적용 (Animal -> Dog)
    IComparable<Dog> dogComp = aniComp; 
    dogComp.CompareTo(new Dog());
    ```


  * 안전한 이유 : 실제 객체(`aniComp`)는 `Animal`을 처리할 수 있으므로,\
    `Animal`의 자식인 `Dog`도 처리할 수 있다.

  * ❓만약 공변성이 허용된다면?
    ```csharp
    class AnimalComparer : IComparable&lt;Animal&gt;
    {
        public int CompareTo(Animal other) {return other.;
    }
    
    class DogComparer : IComparable&lt;Dog&gt;
    {
        public int CompareTo(Dog other); // 내부적으로 비교에 dog에만 있는 속성 사용
    }
    
    IComparable&lt;Animal&gt; aniComp = new DogComparer(); // 공변!
    
    aniComp.CompareTo(new Dog()); // ✅ 
    aniComp.CompareTo(new Cat()); // ❌ InvalidCastException
    ```

