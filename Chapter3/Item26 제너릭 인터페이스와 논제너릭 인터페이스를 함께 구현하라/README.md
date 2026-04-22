## 26. 제너릭 인터페이스와 논제너릭 인터페이스를 함께 구현하라

### 제네릭만 사용했을 때의 문제

* 제네릭 도입 이전에 작성된 코드와 라이브러리가 여전히 존재하기 때문에, 기존 코드와의 호환성을 위해 논제너릭 인터페이스도 함께 구현해야 한다.

* 다음 세 가지에 대해 비제너릭 방식을 지원해야 한다.

  * 클래스 및 인터페이스

  * `public` 속성

  * 직렬화 대상 요소

 

### 구현 패턴

* 제네릭 버전과 논제너릭(`object` 타입 기반) 버전을 모두 구현한다.

* 논제너릭 메서드는 명시적 인터페이스로 구현한다.

  → 인터페이스 참조를 통해서만 호출 가능하게 해, 일반 코드에서는 비제너릭 버전에 대한 접근을 숨긴다.

* 논제너릭 메서드 내부에서는 `object` 타입 기반으로 직접 로직을 구현해도 되고, 제너릭 타입으로 형변환한 뒤 제너릭 메서드를 재사용해도 된다.

* 예시

  ```csharp
  public interface IComparable<T>
  {
  	int CompareTo(T other);
  }
  ```

  ```csharp
  public class Name : IComparable<Name>, IComparable 
  {
      public string First { get; set; }
      public string Last { get; set; }
      public string Middle { get; set; }
  
      // 제너릭 (IComparable<Name>의 멤버)
      public int CompareTo(Name other) 
      {
  		// ...
  		// (생략) 
  		// ...
          return Comparer<string>.Default.Compare(Middle, other.Middle); 
      }
  
      // 비제너릭 (IComparable 멤버)
      int IComparable.CompareTo(object obj)
      {
          if (obj.GetType() != typeof(Name))
          {
              throw new ArgumentException("Argument is not a Name object");
          }
          return this.CompareTo(obj as Name); // 제너릭 메서드 재사용
      }
  }
  
  ```

  ```csharp
  // 사용: 인터페이스 참조를 통해 명시적으로 호출해야 한다.
  if (((IComparable)c1).CompareTo(c2) > 0) 
  	Console.WriteLine("Customer one is greater");
  ```

