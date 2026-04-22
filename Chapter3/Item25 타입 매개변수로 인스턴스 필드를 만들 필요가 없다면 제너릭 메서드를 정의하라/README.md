## 25. 타입 매개변수로 인스턴스 필드를 만들 필요가 없다면 제너릭 메서드를 정의하라

### 제너릭 클래스로 구현했을 때의 단점

* 제약 조건이 광범위하다.

  * 클래스 레벨에서 제약 조건을 걸어야 하므로, 모든 메서드가 동일한 제약 조건을 따라야 한다.

  * 제약 조건의 범위가 넓어질수록 코드 수정과 관리를 까다로워진다.

* 메서드를 호출할 때마다 타입 인수를 명시적으로 지정해야 해서 번거롭다.

* 구체적인 타입에 최적화된 다른 함수가 있더라도 컴파일러가 선택하지 못해 비효율적이다.

* 예시

  * 정의

    ```csharp
    public static class Utils<T> {
        public static T Max(T left, T right) 
    		=> Comparer<T>.Default.Compare(left, right) < 0 ? right : left;
        public static T Min(T left, T right) 
    	    => Comparer<T>.Default.Compare(left, right) < 0 ? left : right;
    }
    ```

  * 사용

    ```csharp
    double d1 = 4;
    double d2 = 5;
    double max = Utils<double>.Max(d1,d2); // 호출 시 번거로움
    
    // 숫자 타입에는 이미 Math.Min()/Max()라는 최적화된 내장 함수가 있는데도,
    // Comparer<T>를 사용해 매번 런타임에 인터페이스 구현 여부를 확인한다.
    ```


### 제너릭 메서드로 구현했을 때의 장점

* 제약 조건이 유연하다.

  * 각 메서드별로 필요한 제약 조건을 독립적으로 설정할 수 있다.

  * 코드를 수정할 때 해당 메서드의 제약 조건만 고려하면 된다. → 유지보수성이 높다.

* 호출시 `T`를 명시할 필요가 없어 호출이 편리하고 코드가 간결해진다.

  <figure><p>컴파일러가 전달된 타입의 인자를 보고 <code>T</code>를 자동으로 추론한다.</p></figure>

* 나중에 더 효율적인 메서드가 추가되면, 기존 호출 코드를 수정하지 않아도 컴파일러가 알아서 새로운 메서드로 연결한다.

  <figure><p>컴파일러는 제네릭 버전보다 구체적인 타입이 명시된 메서드를 우선적으로 선택한다.</p></figure>

* 예시

  ```csharp
  public static class Utils {
  	public static T Max<T>(T left, T right) 
  		=> Comparer<T>.Default.Compare(left, right) < 0 ? right : left;
  	public static double Max(double left, double right) 
  		=> Math.Max(left, right);
  
  	public static T Min<T>(T left, T right) 
  		=> Comparer<T>.Default.Compare(left, right) < 0 ? left : right;
  	public static double Min(double left, double right) 
  		=> Math.Max(left, right);
  }
  
  ```

  ```csharp
  double d1 = 4;
  double d2 = 5;
  double max = Utils.Max(d1,d2); // 내장 함수(Math.Max) 사용
  
  string foo = "foo";
  string bar = "bar";
  stirng sMax = Utils.Max(foo,bar); // 제너릭 메서드 사용
  
  ```


### **제너릭 클래스를 사용해야 할 경우 (예외적)**

1. 클래스 내에 타입 매개변수로 주어진 타입으로 내부 상태를 유지해야 하는 경우

2. 제네릭 인터페이스를 구현하는 클래스를 만들어야 할 경우
