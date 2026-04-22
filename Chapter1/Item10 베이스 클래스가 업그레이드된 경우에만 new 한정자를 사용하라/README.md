## 10. 베이스 클래스가 업그레이드된 경우에만 `new` 한정자를 사용하라

### 가상 메서드vs 비가상 메서드

* C#에서 파생 클래스가 부모 클래스의 메서드를 재정의하는 방식은 크게 두 가지로 나뉩니다.

* 가상 메서드

  * 부모 클래스의 `virtual` 메서드를 `override`를 통해 재정의

* 비가상 메서드

  * 부모 클래스의 일반 메서드를 `new`를 통해 재정의

  * `new` 키워드를 사용하지 않는 재정의는 메서드 숨김(shadowing)에 대한 경고 메시지가 뜬다.

* 비교

  <table><tbody><tr><th><p><strong>구분</strong></p></th><th><p><strong>가상 메서드</strong></p></th><th><p><strong>비가상 메서드</strong></p></th></tr><tr><td><p><strong>목적</strong></p></td><td><p>기본 클래스의 동작을 확장하거나 변경 (다형성 구현)</p></td><td><p>기본 클래스의 메서드를 <strong>숨김</strong> (Shadowing)</p></td></tr><tr><td><p><strong>바인딩 시점</strong></p></td><td><p>동적 바인딩 (런타임)</p></td><td><p>정적 바인딩 (컴파일 타임)</p></td></tr><tr><td><p><strong>호출 기준</strong></p></td><td><p>실제 생성된 객체(Instance)의 타입</p></td><td><p>변수가 선언된 참조(Reference)의 타입</p></td></tr><tr><td><p><strong>관계</strong></p></td><td><p>"파생 클래스는 기본 클래스의 일종이다" (Is-a)</p></td><td><p>"이름만 같을 뿐 서로 다른 메서드이다"</p></td></tr></tbody></table>

 

### `new` 한정자의 문제

* 동일한 객체임에도 불구하고 참조하는 변수에 따라 다른 메서드가 호출되는, 동작 불일치 문제가 발생할 수 있다.

  ```csharp
  public class MyClass
  {
      public void MagicMethod()
      {
  		Console.WriteLine("Parent"); 
  	}
  }
  
  public class MyOtherClass : MyClass
  {
      // 부모의 MagicMethod를 숨기고 새로 정의
      public new void MagicMethod() 
      {
  		Console.WriteLine("Child"); 
      }
  }
  
  MyOtherClass c = new MyOtherClass();
  MyClass cl = c; // 부모 타입으로 캐스팅 
  
  c.MagicMethod();  // 출력: "Child"  (참조가 자식 타입)
  cl.MagicMethod(); // 출력: "Parent" (참조가 부모 타입) 
  //🚨 실제 타입은 모두 MyOtherClass인데 동작 불일치 발생!
  
  ```

  → 개발자들에게 혼란을 줘 가독성/유지보수성을 떨어뜨린다.

 

### `new` 한정자를 사용해야 하는 경우 (예외적인 상황)

* 상황 가정

  * 이미 널리 사용되고 있는 베이스 클래스의 메서드를 재정의하여 완전히 새로운 베이스 클래스로 업그레이드해야 하는 상황이다.

  * 이때, 업그레이드된 새로운 베이스 클래스에만 존재하는 메서드가 기존의 베이스 클래스에 추가됐다고 가정해보자.

* 문제

  * 동일한 이름의 메서드가 정의되어 있기 때문에, 섀도잉 문제가 발생한다.

* 해결

  1. 새로운 클래스 메서드의 이름을 변경한다.

     <figure><p>🚨 그러나 새로운 베이스 클래스가 이미 배포되어 널리 사용되고 있는 경우, 해당 클래스를 사용하고 있던 모든 코드들이 깨지는 문제가 발생한다.&nbsp;</p></figure>

2. `new` 한정자를 사용한다.\
   → 기존의 코드들을 수정할 필요 없이 호환성을 유지할 수 있다.

   <figure><p>🚨 그러나 일부 사용자가 예전 베이스 클래스의 메서드를 계속 사용할 경우, 장기적인 관점으로는 좋지 않다.&nbsp;<br>(차라리 이름 변경하는 게 나을 수도 있다.)</p></figure>

 
