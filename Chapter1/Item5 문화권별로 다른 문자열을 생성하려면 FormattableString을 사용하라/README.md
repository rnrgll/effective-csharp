## 5. 문화권별로 다른 문자열을 생성하려면 `FormattableString`을 사용하라

### 문제점

* 문자열 보간은 코드가 실행되는 그 순간, 문자열이 컴퓨터에 설정된 언어로 확정된다.\
  이후 다른 문화권의 문자열로 바꿀 수 없다. 

* 예시 

  ```csharp
  string s = $"It’s the {DateTime.Now:D}";
  ```

 

### 해결: `FormattableString`

* `FormattableString`은 보간 문자열의 포맷 문자열과 인자 목록을 분리된 상태로 보관하는 타입이다.

  * 문자열 생성을 지연할 수 있다.

  * 문화권을 나중에 명시적으로 지정할 수 있다.

  * → 동일한 보간 문자열로 나라별 언어 설정에 맞춰서 나타낼 수 있다.

* 사용 방법

  * 보간 문자열은 문맥에 따라 `string`이 아니라 `FormattableString`으로 취급될 수 있다.

    ```csharp
    var third =$"It’s the {DateTime.Now:D}";
    ```
    * 이 값을 `FormattableString`을 받는 메서드로 전달하면 문자열 생성을 제어할 수 있다.


* 예제 

  ```csharp
  public static string To German(FormattableString src)
  {
  return string.Format(
          System.Globalization.CultureInfo.CreateSpecificCulture("de-DE"),
          src.Format,
          src.GetArguments());
  }
  
  var third = $"It’s the {DateTime.Now:D}";
  string german = ToGerman(third);
  ```

 
