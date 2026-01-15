

# 46\. **리소스 정리를 위해 using과 try/finally를 활용하라**

`IDisposable`을 구현한 타입을 사용할 때는, 리소스 해제를 위해 **반드시 `Dispose()` 호출이 보장**

되어야 한다.

이를 누락하면 핸들/커넥션 누수, 성능 저하 같은 문제가 발생한다.

## 46.1 문제 상황

가장 단순하게 리소스 정리를 하는 방법은 마지막에 `Dispose()` 를 직접 호출하는 것이다.

```csharp
public void ExecuteCommnad(string connString, string commandString)
{
    var myConnection = new SqlConnection(connString);
    var mySqlCommand = new SqlCommand(commandString, myConnection);

    myConnection.Open();
    mySqlCommand.ExecuteNonQuery(); // 여기서 예외 발생 가능

    mySqlCommand.Dispose();  // 예외가 발생하면 여기까지 못 옴 → 누수
    myConnection.Dispose();  // 예외가 발생하면 여기까지 못 옴 → 누수
}

```

중간에 예외가 발생하면 이후 줄이 실행도지 않아서 `Dispose()` 가 호출되지 않는다.

정상적인 흐름일 때만 정리가 될 수 있다.

## 46.2 `try/finally`로 `Dispose()` 보장하기

위의 문제 상황을 해결하는 기본적인 방법은 `try/finally` 를 사용하는 것이다.

`try` 안에서 어떤 예외가 발생하든 `finally`는 실행되므로, 정리 코드를 **반드시 수행**할 수 있다.

```csharp
public void ExecuteCommnad(string connString, string commandString)
{
    SqlConnection myConnection = null;
    SqlCommand mySqlCommand = null;

    try // 여기서 에러 발생시 마지막 finally에서 리소스 정리
    { 
        myConnection = new SqlConnection(connString); 

        try  // 여기서 에러 발생시 다음 finally에서 리소스 정리
        {
            mySqlCommand = new SqlCommand(commandString, myConnection);
            myConnection.Open();
            mySqlCommand.ExecuteNonQuery();
        }
        finally
        {
            if (mySqlCommand != null)
                mySqlCommand.Dispose();
        }
    }
    finally
    {
        if (myConnection != null)
            myConnection.Dispose();
    }
}

```

## 46.3 `using` 구문 = `try/finally` 자동 생성

매번 `try/finally` 를 사용하는 대신 `using` 구문을 사용하면 간단하게 해결할 수 있다.

### using 구문

**`IDisposable`** 인터페이스를 구현한 객체의 리소스를 자동으로 해제하는 역할을 한다.

*   스코프가 벗어날 때 `Dispose()` 가 반드시 호출된다.

```csharp
using (StreamReader reader = File.OpenText("file.txt"))
{
    string line;
    while ((line = reader.ReadLine()) is not null)
    {
        // 파일 처리
    }
} // 블록을 벗어나면 자동으로 Dispose() 호출

```

*   C# 컴파일러가 자동으로 `try/finally` 패턴을 생성한다.

```csharp
public void ExecuteCommnad(string connString, string commandString)
{
    using (SqlConnection myConnection = new SqlConnection(connString))
    using (SqlCommand mySqlCommand = new SqlCommand(commandString, myConnection))
    {
        myConnection.Open();
        mySqlCommand.ExecuteNonQuery();
    }
}
```

→ 컴파일 결과로 위의 try/finally 예시 코드와 유사한 IL이 나온다.

## 46.4 `using + as IDisposable` 패턴

### `using`은 `IDisposable` 타입에 대해서만 사용 가능하다

`using` 은 **컴파일 타임에 대상이 `IDisposable` 인터페이스를 지원하는 경우에만 사용할 수 있다.**

그렇지 않은 타입에 대해서는 사용 시 컴파일 에러가 발생한다.

`IDisposable` 인지 타입이 애매한 객체인 경우 `as IDisposable` 로 캐스팅하여 처리할 수 있다.

```csharp
using (obj as IDisposable)
{
    // ...
}

```

*   `obj`가 `IDisposable`을 구현하면 ⇒ `Dispose()` 호출됨
*   구현하지 않으면 ⇒ `obj as IDisposable`이 `null`이므로 아무 일도 하지 않는다. (오류 없음)
