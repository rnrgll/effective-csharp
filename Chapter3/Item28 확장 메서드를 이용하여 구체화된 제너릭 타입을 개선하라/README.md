## 28. 확장 메서드를 이용하여 구체화된 제너릭 타입을 개선하라

> 제네릭 클래스가 특정 타입으로 구체화될 때, 기능을 추가하고 싶다면 상속 대신 확장 메서드를 사용하라

### 상속 대신 확장 메서드를 사용해야 하는 이유

* 기존 코드를 수정하지 않고도 새로운 기능을 추가할 수 있다.

* 타입 계층 구조가 복잡해지지 않는다.

### 예시

* `IEnumerable<T>`

  * 타입 매개변수로 특정 숫자 타입이 전달되는 경우, 이에 특화된 메서드를 사용할 수 있다.

  * `int` 타입의 확장 메서드 예시

    ```csharp
    public static class Enumerable
    {
        public static int Average(this IEnumerable<int> sequence);
        public static int Max(this IEnumerable<int> sequence);
        public static int Min(this IEnumerable<int> sequence);
        public static int Sum(this IEnumerable<int> sequence);
    }
    ```

  * 기존 컬렉션 타입을 수정하거나 상속하지 않으면서도, `List<int>`, `int[]`, LINQ 결과 등 모든 `IEnumerable<int>`에 적용 가능한 추가 기능을 사용할 수 있다.

* `CustomerList` vs. `IEnumerable<Customer>`

  * 상속을 사용한 예시

    ```csharp
    public class CustomerList : List<Customer> { 
    	public void SendEmailCoupons(Coupon specialOffer); 
    	public static IEnumerable<Customer> LostProspects(); 
    }
    ```

    * [<img src="data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==" width="1" height="1">](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)새로운 클래스 타입이 추가되었다.


    * `CustomerList`는 구체적인 `List` 클래스를 상속받았기 때문에, 더이상 이터레이터 메서드를 사용할 수 없다.

  * 확장 메서드를 사용한 예시

    ```csharp
    public static void SendEmailCoupons(this IEnumerable<Customer> customers, Coupon specialOffer); 
    public static IEnumerable<Customer> LostProspects(this IEnumerable<Customer> targetList);
    ```

  
 
