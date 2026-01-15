# 11.  .NET 리소스 관리에 대한 이해

.NET 프로그램은 **관리 환경(Managed Environment)** 에서 실행되며,

이 환경에서는 **메모리와 주요 리소스를 CLR(Common Language Runtime)** 이 관리한다.

따라서 C# 개발자는 메모리 관리와 GC의 동작 방식을 정확히 이해하는 것이 중요하다.


## 11.1 GC(Garbage Collector)

GC는 **관리되는 힙(Managed Heap)** 에 할당된 객체의 생명주기를 관리한다.

네이티브 언어(C, C++)와 달리 .NET에서는 다음과 같은 문제를 개발자가 직접 처리하지 않아도 된다.

*   메모리 누수
*   댕글링 포인터 (이미 해제된 메모리를 가리키는 포인터)
*   초기화되지 않은 포인터
*   복잡한 객체 참조 관계에서의 해제 순서 문제
    *   순환 참조 문제 : 객체들이 서로를 참조하여 연결 고리가 끊어지지 않는 상태
 


## 11.2 GC의 기본 동작 원리 (Mark & Compact 알고리즘)

**Mark → Sweep → Compact** 흐름으로 동작한다.

최상위 루트 객체에서 출발해서 여러 객체 사이의 연관 관계를 파악하여 더 이상 사용되지 않는 객체를 자동으로 제거하는 방식이다.

### 11.2.1 GC Root

GC는 살아 있는 객체(Reachable Objects)와 죽은 객체(Unreachable Objects)를 구분하기 위해 ‘어디선가 참조되고 있는가’를 판단한 기준이 필요하다. 이 기준점이 바로 GC Root 이다. 

*   스택에 있는 지역 변수
*   정적(static) 필드
*   Finalizer Queue에 있는 객체



### 11.2.2 Mark / Sweep / Compact 단계
<img width="835" height="594" alt="image" src="https://github.com/user-attachments/assets/981371f6-90e0-40de-8cc2-eb6a1a520c7e" />

출처 : [https://blog.naver.com/lth1044/222337611824](https://blog.naver.com/lth1044/222337611824)

#### Mark 단계

*   GC Root에서 출발한다.
    
*   참조 체인을 따라가며 **도달 가능한 모든 객체에 마킹한다.**
    
*   객체 스스로 참조 여부를 관리할 필요 없다.
    

#### Sweep 단계

*   Mark 되지 않은 객체 = **더 이상 접근 불가능**
*   해당 객체들을 Garbage로 판단하여 제거 대상에 포한다.

#### Compact 단계

*   여러 객체가 할당 해제되고 나면 빈 공간이 세그먼트 내 여러 곳에 산재될 수 있다. 이 경우 전체적인 빈 공간은 충분하지만, 큰 객체를 할당할 수 있는 연속된 메모리 공간이 없어 객체를 할당할 수 없는 문제가 발생한다.
*   이를 해결하기 위해 살아남은 객체들을 한쪽으로 밀어 정렬하여 비어 있는 메모리를 하나로 모아 **메모리 단편화(Fragmentation)을 해결한다.**

<img width="595" height="176" alt="image" src="https://github.com/user-attachments/assets/5f400f62-8519-4022-83c3-1dd36444d499" />

출처 : [https://siahn95.tistory.com/108](https://siahn95.tistory.com/108)


## 11.3 GC 성능 최적화 : Generation 개념

GC는 모든 객체를 동일하게 취급하지 않는다.

객체의 **생존 시간에 따라 세대(Generation) 로 나누어 관리**한다.

### 11.3.1 세대 구분
| 세대 | 설명 |
| --- | --- |
| 0세대 | 가장 최근에 생성된 객체 = GC 이후에 생성된 객체 |
| 1세대 | GC 이후에도 살아남은 객체 |
| 2세대 | 여러 GC를 거쳐 장기간 살아남은 객체 |


### 11.3.2 GC 동작 전략

대부분의 객체는 **짧은 생명주기**를 갖기 때문에 최근에 생성된 객체를 위주로 GC를 수행하는 것이 효율적이다.

<img width="966" height="670" alt="image" src="https://github.com/user-attachments/assets/cc347b63-7405-49ef-bd97-013ca11dc050" />

*   GC0 : 가장 자주 실행
*   GC1 : 0세대가 꽉 찼을 때 (보통 10번 중 1번)
*   GC2 : 메모리가 많이 필요할 때 제한적으로 수행 (약 100번 중 1번)

```text
Finalizer 객체와 Generation

- Finalizer를 가진 객체는 **즉시 제거 불가**
- 최소 1세대까지 승격됨
- 따라서 메모리에 오래 머물게 됨 → GC 부담 증가
```


## 11.4 GC 성능 최적화 : 점진적 GC(Incremental GC)

GC 작업을 여러 개의 작은 슬라이스로 분할하여 여러 프레임에 걸쳐 분산시키는 방법이다.

예를 들어, 기존에 한 프레임에서 100ms가 걸리던 GC 작업을 10개의 슬라이스로 나누어 10개 프레임에 걸쳐 각각 10ms씩 처리한다.

<strong>전체 GC 소요 시간은 동일하지만, 각 프레임의 중단 시간이 짧아져 GC Spike가 줄어들고</strong> 부드러운 애니메이션을 유지할 수 있다.

### 11.4.1 Unity GC

Unity는 Boehm GC를 사용한다.

#### 특징

*   비세대적(Non-Generational) 방식 : 세대별 관리를 하지 않고, 모든 객체를 동일하게 취급한다.
*   비압축 (Non-Compacting) 방식 :  메모리 압축(Compaction)을 수행하지 않는다. GC가 사용하지 않는 메모리를 해제한 후에도 힙의 단편화(fragmentation)를 해결하기 위해 객체를 재배치하지 않는다.
    > **<p>Unity가 비압축 방식을 사용하는 이유</p>** Unity는 C# 스크립트와 C++ 네이티브 엔진이 연동되는 구조를 가진다. C++ 코드가 C# 객체의 메모리 주소를 직접 참조하는 경우가 많다. 만약 GC가 메모리 압축을 수행하여 객체를 이동시키면, 네이티브 코드에서 보관 중이던 포인터가 더 이상 유효하지 않게 되어 치명적인 오류가 발생할 수 있다. 또한, 메모리 압축은 다수의 객체를 이동시키는 고비용 작업으로, 긴 Stop-the-World 시간을 유발할 수 있어 실시간 프레임 유지가 중요한 게임 환경에 적합하지 않다.
*   Stop-the-World 방식 : GC가 시작되면 프로그램 전체가 멈추고 GC가 완료될 때까지 기다려야 한다. 처리해야 할 메모리 양과 플랫폼에 따라 게임이 수백 밀리초나 멈춰 있을 수 있으며, 이를 GC Spike라고 한다.
*   점진적 GC(Incremental Garbage Collection) 방식 : GC Spike를 완화하기 위해 Unity 2019.1 부터 도입되었다. Boehm GC가 incremental mode로 실행된다.


### 11.4.2 .NET GC

.NET의 가비지 컬렉터는 Unity의 Boehm GC와는 다른 알고리즘으로 작동한다.

#### 특징

*   세대별 GC 방식
*   Mark-Sweep-Compact 알고리즘 : Unity의 Boehm GC가 Mark-Sweep만 수행하는 것과 달리, .NET GC는 Compact 단계를 포함한다.

### 11.4.3 Unity GC vs .NET GC 
| **특징** | **Unity (Boehm GC)** | **.NET GC** |
| --- | --- | --- |
| **세대 관리** | 비세대적 (전체 힙 스캔) | 세대별 (Gen 0, 1, 2) |
| **압축(Compact)** | 압축하지 않음 | 압축 수행 (단편화 방지) |
| **수집 방식** | Stop-the-world | 동시/백그라운드 가능 |
| **알고리즘** | Mark-Sweep | Mark-Sweep-Compact |
| **메모리 단편화** | 발생 가능 (심각) | 압축으로 방지 |


## 11.5 비관리 리소스

데이터베이스 연결, 시스템 객체(GDI+ 객체, COM 객체)와 같은 비관리 리소스는 **개발자가 직접 생명주기를 관리해야 한다.**

*   네트워크 소겟, 파일 핸들, 그래픽/오디오 관련 네이티브 리소스 등
    
*   GDI+ 객체 : Windows의 그래픽 처리를 위한 객체지향 라이브러리로, 도형, 이미지 텍스트 등 다양한 그래픽을 그리는데 사용된다.
    
    → 쉽게 말해, 화면에 그림을 그리기 위한 시스템 자원을 객체 형태로 제공하는 것
    
*   COM 객체 : 마이크로소프트의 컴포넌트 기술 표준. 여러 프로그래밍 언어와 환경에서 재사용 가능한 소프트웨어 컴포넌트를 만들 수 있게 해준다. → 다른 프로그램 또는 시스템 컴포넌트와 상호작용하기 위한 규격 및 객체
    

비관리 리소스 생명주기 관리를 위해 .NET Framework는 다음 두 가지 메커니즘을 제공한다.

*   finalizer
*   IDisposable

### 11.5.1 finalizer (소멸자)

비관리 리소스에 대한 <span class="fontColorRed"><strong>해체 작업이 반드시 수행되도록 보장</strong></span>하기 위한 방어적 메커니즘이다.

가비지 수집기가 **클래스 인스턴스를 수집할 때, 최종 정리 작업을 수행하기 위해 자동으로 호출된다.**

#### 특징

*   구조체에서 정의할 수 없음. **클래스에서만 사용됨.**
*   클래스에는 **하나의 소멸자**만 가질 수 있음.
*   상속하거나 오버라이드 할 수 없음
*   **개발자가 직접 호출할 수 없음 <span class="fontColorRed">→ GC 메커니즘에 의해 자동으로 호출됨</span>**
*   접근 한정자를 사용할 수 없으며, 매개변수를 가질 수 없음

#### 예시

```csharp
class Car
{
   ~Car() // finalizer
   {
	   // 정리
   }
}
```

참고 : [https://learn.microsoft.com/ko-kr/dotnet/csharp/programming-guide/classes-and-structs/finalizers](https://learn.microsoft.com/ko-kr/dotnet/csharp/programming-guide/classes-and-structs/finalizers)


#### finalizer 객체의 GC 과정

C# 클래스에 Finalizer가 존재하면, 해당 객체의 생성자 호출 시 **`Finalization queue` 에 레퍼런스를 추가**.

1.  C# 클래스에 Finalizer가 존재하면, 해당 객체의 생성자 호출 시 **`Finalization Queue` 에 레퍼런스가 등록된다.**
2.  GC 수행시 (GC Thread)
    *   Finalization Queue에 레퍼런스가 없는 객체는 Garbage로 판단되어 제거된다.
    *   Finalization Queue에 레퍼런스가 있는 객체는 즉시 제거되지 않고 **`Freachable Queue`라는 다른 큐로 이동된다.**
3.  GC 이후, **Finalizer Thread가 `Freachable Queue`의 객체를 꺼내 Finalizer를 호출**하여 비관리 리소스를 정리한다. **(Finalizer Thread)**
4.  Finalizer 실행이 완료된 이후, 다음 GC 사이클에서 해당 객체의 메모리가 비로소 회수된다.


#### 단점

*   객체 해제가 지연된다
    
    Finalizer를 가진 객체는 즉시 메모리에서 제거되지 않으며, Finalizer 실행 이후 **다음 GC 사이클까지 살아남기 때문에 최소 두 번의 GC를 거쳐야 완전히 제거된다.**
    
*   메모리 점유 시간이 길다
    
    Finalizer 실행 전까지 객체가 메모리에 유지되므로, 메모리 회수가 지연되어 전체 메모리 사용량이 증가할 수 있다.
    
*   **추가적인 오버헤드**가 발생한다.
    
    Finalization Queue 및 Freachable Queue 관리 비용과 Finalizer Thread로의 작업 이관 과정에서 추가적인 오버헤드가 발생한다.
    
*   스레드 전환 비용이 발생한다
    
    Finalizer는 GC Thread가 아닌 별도의 Finalizer Thread에서 실행되므로, 큐 처리 및 스레드 전환에 따른 성능 비용이 발생한다.
    
*   정확한 해제 시점을 보장하지 못한다.
    
    Finalizer는 언젠가는 호출되지만, 개발자가 원하는 시점에 호출되도록 제어할 수는 없다.

    

### 11.5.2 IDisposable

finalizer는 비관리 리소스를 **최종적으로 해제할 수 있는 마지막 수단**이지만, `IDisposable` 인터페이스와 **표준 Dispose 패턴**을 활용하면 가비지 수집 과정의 지연을 방지하고 리소스를 더 안전하게 관리할 수 있다.

(\* IDisposable에 대한 상세 내용은 이후 항목에서 설명)
