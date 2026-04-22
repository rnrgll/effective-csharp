## 6. `nameof()` 연산자를 적극 활용하라

### `nameof` 연산자의 의미

* 변수, 타입(Type), 멤버(Member)의 심볼(Symbol) 이름을 컴파일 타임 상수 문자열로 반환한다.

* 컴파일러가 해당 심볼의 이름을 실제 문자열 리터럴로 치환한다.

  ```csharp
  // 코드 작성
  Console.WriteLine(nameof(System.Collections.Generic));
  Console.WriteLine(nameof(List<int>.Count));
  
  // 컴파일 후 (IL/실제 동작)
  Console.WriteLine("Generic");
  Console.WriteLine("Count");
  
  ```


* 📌 항상 로컬 이름(Local Name)을 문자열로 반환한다.

  * 완전히 정규화된 이름을 사용하더라도 항상 로컬 이름으로 반환된다.

  * `System.Int.MaxValue` ❌ / `MaxValue` ✅ 

 

### 등장 배경

* 과거에는 식별자(변수명, 함수명 등)를 참조할 때 문자열을 직접 입력(하드코딩)해서, 다음과 같은 문제들이 발생했다.

  * 문자열은 단순 텍스트이므로 타입 정보가 손실된다.

  * 변수 이름을 바꾸면 하드코딩된 문자열은 자동으로 수정되지 않는다.

  * 오타가 발생해도 컴파일러가 잡아내지 못하고 런타임 에러로 이어진다.

 

### 장점

* 리팩토링 자동화 지원

  * 이름을 바꾸면 `nameof` 내부의 심볼 이름도 함께 변경된다.

* 컴파일 타임 유효성 검사

  * 존재하지 않는 이름을 쓰거나 오타가 나면 컴파일 에러가 발생하여 실수를 방지한다.

* 성능 이점

  * 리플렉션과 달리 컴파일 시점에 문자열로 변환되므로 런타임 오버헤드가 없다.

 

### 주요 활용 예시

* 예외 처리

  * 매개변수 검사 시 인자의 이름을 문자열로 전달해야 할 때 유용하다.

  ```csharp
  public static void ExceptionMessage(object thisCantBeNull)
  {
      // ❌ 나쁜 예 (문자열 하드코딩)
      if (thisCantBeNull == null)
          throw new ArgumentNullException("thisCantBeNull"); 
          // 매개변수명이 바뀌면 이 문자열은 틀린 정보가 됨
  
      // ✅ 좋은 예 (nameof 사용)
      if (thisCantBeNull == null)
          throw new ArgumentNullException(nameof(thisCantBeNull));
          // 매개변수명이 바뀌면 같이 바뀜
  }
  
  ```


* 로그 메시지

  * 메서드나 변수 이름을 하드코딩하면 리팩토링 시 로그 내용이 실제 코드와 어긋날 수 있기 때문이다.
