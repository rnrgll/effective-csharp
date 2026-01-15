


# 44\. 바인딩된 변수는 수정하지 말라

람다나 쿼리(클로저)가 캡처하는 **바인딩된 변수**는 값이 아니라 \*\*변수 그 자체(저장 위치)\*\*를 참조한다.

따라서 람다 정의 이후에 그 변수를 수정하면, 람다 실행 결과가 **의도와 다르게 통째로 바뀔 수 있다.**

## 44.1 바인딩된 변수 수정 시 발생하는 문제 / 원인

### 문제 상황

```csharp
// 기대 동작
TabControlButton[0].Click +=delegate { Change_Tab(0); };
TabControlButton[1].Click +=delegate { Change_Tab(1); };
TabControlButton[2].Click +=delegate { Change_Tab(2); };
TabControlButton[3].Click +=delegate { Change_Tab(3); };
TabControlButton[4].Click +=delegate { Change_Tab(4); };

// 구현한 코드
for (int i =0; i < TabControlButton.Length; i++)
{
    TabControlButton[i].Click +=delegate { Change_Tab(i); };
}
```

실제 실행 시 제대로 동작하지 않음 → 모든 버튼에서 Change\_Tab(5)이 실행됨

<img width="763" height="374" alt="image" src="https://github.com/user-attachments/assets/dc74eca0-394d-4332-bd4a-ac3144f0c384" />

출처 : [https://doublsb.tistory.com/73](https://doublsb.tistory.com/73)

### 원인

*   람다/쿼리는 보통 **지연 실행**된다.
    
*   클로저는 변수의 ‘값’이 아니라 **같은 변수**를 공유한다.
    
    **값을 캡처하는 것이 아니라 변수를 캡쳐한다. 즉, 변수의 저장 위치를 참조한다.**
    
*   즉, 람다를 만들 때 변수의 값이 고정되고, 이를 사용하는게 아니라 **람다를 실행할 때 현재 변수 값이 사용된다.**
    

⇒ 따라서 람다 밖에서 변수를 바꾸면 람다 결과도 함께 바뀐다.

### 바인딩 된 변수 사용시 규칙

1.  캡처된 변수는 불변처럼 취급한다.
    
2.  값을 고정하고 싶다면 지역 복사본을 만들어 캡처한다.
    
    ```csharp
    for (int i =0; i < TabControlButton.Length; i++)
    {
    		int j = i;// 지역 복사본
    		TabControlButton[i].Click +=delegate { Change_Tab(j); };
    }
    ```
    
    *   `j`는 매 반복마다 새로 생성된다.
    *   각 람다는 **자기만의 j**를 캡처한다.
    *   값이 고정되어 의도대로 동작한다.
3.  람다 안에서 부수 효과(++ 등)를 만들지 않는다.
    

## 44.2 람다가 항상 클로저는 아니다.

컴파일러는 쿼리 표현식과 람다 표현식을 **정적 델리게이트나 인스턴스 델리게이트 혹은 클로저로 변환**한다.

이 중 어떤 것을 선택하느냐는 람다 본문의 코드가 어떻게 작성됐냐에 따라 결정도니다.

*   지역변수/매개변수 접근이 없으면 → **정적 메서드 + 정적 델리게이트**
    
    ```csharp
    int[] someNumbers = {0, 1, 2, 3, 4, 5, 6, 7, 8. 9, 10};
    var answers = from n in someNumbers
    							select n * n;
    ```
    
    ```csharp
    //select n * n 은 정적 델리게이트로 변환된다.
    private static int HiddenFunc(int n) => n * n;
    priavte static Func<int, int> HiddenDelegateDefinition;
    ```
    
*   인스턴스 필드에는 접근하지만 지역변수에는 접근하지 않으면 → **인스턴스 메서드 델리게이트**
    
*   지역변수/매개변수까지 접근하면 → **클로저(중첩 클래스)**
    
    ⇒ 이 경우에 클로저가 만들어 진다.

