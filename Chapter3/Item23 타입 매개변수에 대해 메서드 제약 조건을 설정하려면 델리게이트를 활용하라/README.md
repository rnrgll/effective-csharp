## 23. 타입 매개변수에 대해 메서드 제약 조건을 설정하려면 델리게이트를 활용하라

### C#의 기본 제약 조건의 한계

* 제약 조건의 종류가 한정되어 있어, 표현력이 제한적이다.

* 특정 시그니처의 생성자 또는 메서드 구현을 강제할 수 없다.\
  → 필요한 경우 별도의 인터페이스 정의가 필요하다.

###  

### **인터페이스 vs 델리게이트**

* 인터페이스를 통한 제약 조건 정의

  * 사용 방법

    1. 인터페이스 정의

    2. 인터페이스를 상속받은 클래스 및 함수 구현

    3. 인터페이스로 제약 조건을 설정

  * 장점

    * 타입 수준에서 계약을 명확히 표현할 수 있다.

    * 설계 의도가 명확하게 드러난다.

  * 단점

    * 타입의 수가 증가한다.

    * 단순한 연산 하나를 위해 해야할 일이 너무 많아 복잡해진다.

  * 예시

    : 타입 매개변수 `T` 가 `Add` 함수를 구현하길 원하는 경우

    ```csharp
    // 인터페이스 정의
    public interface IAddable<T>
    {
        T Add(T other);
    }
     
    // 특정 클래스에서 기능 구현
    public struct MyInt : IAddable<MyInt>
    {
        public int Value { get; }
        public MyInt(int value)
        {
            Value = value;
        }
        public MyInt Add(MyInt other)
        {
            return new MyInt(this.Value + other.Value);
        }
    }
     
    //제약 조건을 설정
    public static class Example
    {
        public static T Add<T>(T a, T b) where T : IAddable<T>
        {
            return a.Add(b);
        }
    }
    ```

* 델리게이트를 통한 제약 조건 정의

  * 사용 방법

    1. 적절한 메서드의 원형을 고안하고 이를 델리게이트로 정의

    2. 델리게이트의 인스턴스를 제네릭 메서드의 매개변수로 추가

    3. 호출자에서 람다 표현식 또는 메서드 그룹 전달

  * 예시

    : 타입 매개변수 `T` 가 `Add` 함수를 구현하길 원하는 경우

    ```csharp
    public static class Example
    {
        public static T Add<T>(T Left, T Right, Func<T,T,T> AddFunc) => AddFunc(Left,Right);
    }
     
    int a = 6;
    int b = 7;
    int sum = Example.Add(a, b, (x,y) => x + y);
    ```

  * 장점

    * 기존 타입을 수정하지 않고도 사용할 수 있다.

    * 코드가 간결하고 구현하기 편하다.

###  

### 어떤 방식을 선택할 것인가

* 인터페이스 사용

  * 특정 타입에 대한 제약을 명시적으로 드러내야 할 때

  * API 설계 차원에서 기능 계약을 드러내야 할 때

* 델리게이트 사용

  * 특정 메서드나 로직을 쉽게 전달하고자 할 때

  * 기존 타입의 수정을 최소화하면서 새로운 기능을 추가할 때

  * 동작이 상황별로 달라질 수 있을 때
