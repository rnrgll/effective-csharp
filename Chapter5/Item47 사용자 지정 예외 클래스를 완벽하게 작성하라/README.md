
# 47\. 사용자 지정 예외 클래스를 완벽하게 작성하라

## 47.1 사용자 지정 예외 클래스

발생 가능한 예외 상황을 특정하기 위한 사용자 정의 예외 클래스다.

내장된 예외 클래스를 사용하여 예외 처리를 수행할 수 있지만, **특정 예외 상황에 대한 정보를 더 자세히 제공하거나 사용자 지정 예외 처리 로직을 구현해야 할 때** 예외 클래스를 정의하고 사용할 수 있다.

→ ‘이 예외가 우리 프로그램에서 어떤 의미인지’ 를 **타입으로 표현**하고, 처리하는 쪽이 **분기/복구**할 수 있게 **추가 정보(상태)** 를 담는 예외

## 47.2 사용자 지정 예외를 만드는 상황

1.  **서로 다른 예외를 구분하여 다른 대응이 필요할 때**
    
    기본 `Exception`을 잡은 뒤 **메시지 문자열로 분기**하면 다음과 같은 문제가 생긴다.
    
    ```csharp
    try
    {
        LoadSave();
    }
    catch (Exception ex)
    {
        // 메시지/문자열/코드에 의존
        switch (ex.Message)
        {
            case "SAVE_NOT_FOUND":
                CreateNewSave();
                break;
    
            case "SAVE_CORRUPTED":
                ShowCorruptedPopup();
                break;
        }
    }
    
    ```
    
    *   문자열은 컴파일 타임 체크가 불가능 → 오타나 메시지 변경 시 <span class="fontColorBlue"><strong>런타임 오류 발생</strong></span>
    *   공통 라이브러리로 이동하면 메시지 값이 달라짐 → 라이브러리 메시지가 변경되거나, 라이브러리 경계를 넘나들며 <span class="fontColorBlue"><strong>깨질 수 있음</strong></span>
    *   호출 스택이 깊어지면 메시지의 의미를 잃음 → “SAVE\_NOT\_FOUND”가 어떤 계층에서 발생했는지 <span class="fontColorBlue"><strong>추적 어려움</strong></span>
    
    따라서 다음과 같이 필요한 예외에 대해서 사용자 지정 예외 클래스를 만들고, **타입 자체로 분기**할 수 있도록 예외 클래스를 분리하는 것이 맞다.
    
    ```csharp
    catch (SaveNotFoundException)
    {
        CreateNewSave();
    }
    catch (SaveCorruptedException)
    {
        ShowCorruptedPopup();
    }
    
    ```
    
2.  **예외에 추가 정보가 필요할 때**
    
    *   기본 `Exception` 클래스는 단순 메시지만 가지고 있어 상황을 자세히 설명하기 어렵다.
        
    *   이미 존재하는 예외 타입만으로 특화 정보를 담기 부족한 경우, 예외 객체에 필요한 정보를 속성으로 추가한 사용자 지정 예외를 만든다.
        
        : user 아이디, 파일 경로, 에러 코드, 일시적 오류인지 영구적 오류인지 등의 정보
        
    
    ```csharp
    public class SaveCorruptedException : Exception
    {
        public string FilePath { get; }
        public int SlotId { get; }
    
        public SaveCorruptedException(string filePath, int slotId, Exception inner = null)
            : base($"Save file is corrupted. (Path={filePath}, Slot={slotId})", inner)
        {
            FilePath = filePath;
            SlotId = slotId;
        }
    }
    ```
    
    예를 들어, 저장 중 예외가 발생한 상황에 대한 예외 클래스를 만들고 어떤 파일, 슬롯에서 문제가 발생했는지 등의 정보를 담아두면, catch 하는 쪽에서 정보를 기반으로 더 나은 대응을 할 수 있다.
    
3.  **즉시 처리되지 않으면 심각한 문제인 경우**
    
    단순한 입력 오류 수준이 아니라 시스템에 논리적 문제가 있는 경우, 심각한 오류인 경우에는 반드시 별도의 예외 타입으로 구분하는 것이 좋다.
    
    예) 데이터 베이스 정합성 문제(예 : 결제 완료인데 주문 테이블에 주문이 없음, 인벤토리 수량이 음수 등)
    
4.  **저수준 예외를 숨기고 고수준 예외로 재해석할 때**
    
    라이브러리, DB, 네트워크 등에서 발생한 예외를 그대로 노출하면 다음 문제가 생긴다.
    
    *   사용자에게 보여줄 메시지가 없음 (기술적인 내용뿐)
    *   특정 구현(파일 시스템, DB 드라이버 등)에 종속됨
    *   복구 가능성 판단이 어려움
    
    이러한 경우 사용자 지정 예외 타입을 만들어서 더 의미 있는 예외 타입을 만들어서 대응하게 할 수 있다.
    
    (\*47.4에서 추가로 설명)
    

## 47.4 예외 클래스 작성하는 방법

1.  개별 예외 클래스에 대한 고유 책임을 명확히 규정하기
    
2.  모든 예외 클래스의 이름은 Exception으로 끝나기
    
3.  System.Exception클래스 혹은 더 적절한 예외클래스를 상속해서 구현할 것
    
4.  아래 4개의 생성자는 반드시 작성할 것
    
    ```csharp
    //기본 생성자
    public Exception();
    //에러 메시지를 포함하는 생성자
    public Exception(string);
    //에러 메시지와 내부 예외를 포함하는 생성자
    public Exception(string, Exception);
    //입력 스트림을 이용하는 생성자
    protected Exception(SerializationInfo, StreamingContext);
    ```
    

### 47.4.1 예외 클래스 예시

게임의 세이브 파일을 읽는데 JSON 구조가 깨져있거나 버전이 맞지 않을 때 던지는 예외

```csharp
using System;
using System.Runtime.Serialization;

[Serializable]
public class InvalidSaveFormatException : Exception
{
    // 1. 기본 생성자
    public InvalidSaveFormatException()
    {
    }

    // 2. 메시지를 받는 생성자
    public InvalidSaveFormatException(string message)
        : base(message)
    {
    }

    // 3. 메시지 + 내부 예외를 받는 생성자
    public InvalidSaveFormatException(string message, Exception innerException)
        : base(message, innerException)
    {
    }

    // 4. 직렬화(역직렬화) 생성자
    protected InvalidSaveFormatException(SerializationInfo info, StreamingContext context)
        : base(info, context)
    {
    }
}

// 사용
try
{
    var save = LoadSave("save.json");
}
catch (JsonException ex)
{
    throw new InvalidSaveFormatException("세이브 파일이 잘못된 형식입니다.", ex);
}
```

### 47.4.2 InnerException 생성자

*   외부 라이브러리에서 발생한 예외(`SqlException`, `IOException` 등)는 **기술적인 원인 정보는 풍부하지만**, 응용 프로그램 관점에서 **복구 작업에 필요한 의미를 바로 제공하지는 않는다.**
*   이 예외를 그대로 상위 계층으로 전달하면 상위 코드(UI / 도메인 / 게임 로직)가 **라이브러리·인프라 세부사항을 직접 알아야 한다.**
    *   상위 계층이 직접 처리하게 되면 <span class="fontColorRed"><strong>계층 간의 결합이 증가한다.</strong></span>
    *   예를 들어, SqlException (DB 에러)를 상위 계층이 직접 처리하면 DB 기술에 종속된다. 나중에 DB를 변경하거나 저장 방식이 바뀌면 상위 코드의 catch 문에 모두 깨진다.

따라서 다음과 같은 방식이 필요하다.

#### **예외 변환(Exception Wrapping)**

**저수준 에러 vs 고수준 에러**

*   저수준 에러
    *   **기술적인 원인이 중심인 에러**
    *   시스템 / 라이브러리 / 인프라 관점의 오류
    *   예)
        *   `SqlException` : DB 드라이버/서버에서 오류
        *   `IOException` : 파일/스트림 I/O 실패
        *   `SocketException` : 네트워크 소켓 오류
        *   `NullReferenceException` : 객체가 null인데 접근함
    *   응용 프로그램 관점에서 해당 에러가 ‘어떤 의미인지’ 알기 어렵다. **사용자에게 어떤 메시지를 보여야 하는지, 어떻게 해야하는지, 재시도를 해야하는지 등의 판단이 어렵다.**
*   고수준 에러
    *   **응용 프로그램 관점에서 재해석한 오류**
    *   현재 애플리케이션에 무엇이 실패했는 지를 표현
    *   예)
        *   `UserNotFoundException` : 로그인 시 유저가 없음
        *   `InventoryNotEnoughException` : 아이템 소비 불가
        *   `SaveDataCorruptedException` : 저장 데이터가 깨짐
    *   응용 프로그램의 복구 전략을 결정할 수 있는 정보를 포함한다. (재시도 가능 여부, 사용자에게 보여줄 메시지, 로깅/모니터링에 남길 상황 정보) => 일관성 있는 대응이 가능

#### 예외 변환

*   저수준 예외(라이브러리/인프라 예외)를 **catch** 한 뒤,
*   응용 프로그램 관점에서 의미가 분명한 <span class="fontColorRed"><strong>고수준 예외 객체를 새로 생성</strong>한다.</span>
*   이때, **원래 발생한 저수준 예외는 <span class="fontColorRed"> InnerException(프로퍼티)에 저장</span>**하여 함께 전달한다.

→ 상위 계층은 **고수준 예외 타입만 보고** 적절한 대응을 할 수 있고

→ 실제 문제의 기술적 원인은 `InnerException`을 통해 **로그·디버깅 용도로 추적 가능**하다.

##### 예시 코드

```csharp
try
{
    sqlConnection.Open(); //SqlException이 발생
}
catch (SqlException ex)
{
    throw new DatabaseUnavailableException(
        "현재 데이터베이스에 접근할 수 없습니다.",
        ex);
}
```

*   **저수준 예외 (`SqlException`)** : DB 드라이버 / 서버 관점의 기술적 오류로 DB 연결이 실패했는가에 대한 상세 원인 제공
*   **고수준 예외 (`DatabaseUnavailableException`) :** 응용 프로그램 관점의 의미, 현재 프로그램에서 발생한 상태가 어떤지에 대한 정보 제공, 재시도 여부, 사용자 메시지, 서비스 동작 정책 판단에 사용
