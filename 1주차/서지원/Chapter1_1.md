# Chapter1-1 오브젝트와 의존 관계

## DAO (Data Access Object)
DB를 사용해 데이터를 조회하거나 조작하는 기능을 전담하도록 만든 오브젝트를 의미한다. <br>
즉, **Service와 DB를 연결**하며, 실제로 DB에 접근하여 data의 **CRUD 기능을 수행하는 역할**을 한다. 

> 💡JPA에서는 JPARepository를 상속 받는 Repository를 의미한다. 

<br>

### 📌 객체 지향을 통한 설계와 코드는 변화한다. 따라서, 개발자는 객체를 설계할 때 미래의 변화를 어떻게 대비할 것인지 생각해야한다. 

그렇다면 우리는 변경이 일어날 때 어떻게 <span style="color:orange;">**필요한 작업을 최소화**</span>하고 그 <span style="color:orange;">**변경이 다른 곳에 영향을 일으키지 않게**</span> 할 수 있을까?
<br>

### ➡︎ `분리와 확장을 고려한 설계` 

<br>

- 모든 변경과 발전은 <ins>**한 번에 한가지 관심사항에 집중**</ins>한다. 
- 변화는 한가지 관심에 대해 일어나지만 <ins>**그에 따른 작업은 한곳에 집중되어 있지 않는 경우가 많다.**</ins> 

따라서, 관심이 같은 것끼리는 모으고, 관심이 다른 것은 따로 떨어져 있게 해야한다. **[관심사 분리]**


---

### 관심사를 분리 과정

**1. 중복 코드의 메소드 추출**

코드를 수정한 후에는 변경 전과 동일하게 동작하는지 확인하는 과정이 필요하다. **리팩토링과 테스트**

> 💡 **︎공통 기능을 담당하는 메소드**로 `중복된 코드를 뽑아내는 것`을 리팩토링에서 <span style="color:orange;">**메소드 추출 기법**</span>이라고 부른다. 

<br>

**2 - 1. 상속을 통한 확장**   <br> 
특정 관심 부분을 상속을 통해 자식 클래스로 분류한다면 새로운 방법을 적용해야 할 때 상속을 통해 확장해주기만 하면된다.  

- 기능 일부를 추상 메서드나 오버라이딩 가능한 메소드로 만든 후 `자식 클래스`에서 **해당 메소드를 필요에 맞게 구현해서 사용하는 것**
  ➡︎ <span style="color:orange;">**템플릿 메소드 패턴**</span>
- `자식 클래스`에서 **구체적인 오브젝트 생성 방법을 결정하게 하는 것** ➡︎ <span style="color:orange;">**팩토리 메서드 패턴**</span>

> 💡 **디자인 패턴:** <ins>**재사용 가능한 솔루션**</ins>을 의미하며, 간단한 `패턴 이름 언급만으로 설계의 의도와 해결책을 함께 설명`할 수 있는 장점이 있다. 

> 💡 **템플릿 메서드 패턴:** 상속을 통해 <ins>**부모 클래스의 기능을 확장할 때 사용**</ins>하며, 변하지 않는 기능은 부모 클래스에 만들어두고 <ins>**자주 변경할 기능은 자식 클래스에 만드는 것**</ins>을 말한다. 
> <br>
> <br>
> 💡**팩토리 메소드 패턴:** <ins>**객체 생성을 하는 클래스를 따로 두는 것**</ins>을 의미한다. 
> <br> 
> 
> &nbsp;  **[팩토리 메서드, 템플릿 메서드 패턴](https://western-sky.tistory.com/40) - 참고자료**

<br>

🚨
-  <span style="color:#F05750;">**자바는 클래스의 다중 상속을 허용하지 않기 때문에**</span> 상속을 통해 확장을 하면 **서브 클래스는 다른 목적의 기능을 지니는 슈퍼 클래스를 상속 받지 못한다.** 


-   상하위 클래스의 관계는 생각보다 밀접하다. 

<br>


**2 - 2 . 클래스 분리**

위와 같은 방법으로 확장을 하면 결국 두가지 관심사에 대해 긴밀한 결합을 허용하게 된다. 

따라서, 이번에는 완전히 **독립적인 클래스를 만들어 분리**를 진행한다.

```java
public class Dao {
    private ExampleClass exampleClass;

    public Dao() {
        this.exampleClass = new ExampleClass(); // ⚠️개발자는 DAO에서 어떤 클래스를 쓸지 결정해야한다.  
    }
}
```

```java
public class ExampleClass {
    
    private String name = "example";
    
    pulbic int add() {
        ... 
    }
    
    public String getName() {
        return this.name;
    }
}
```

위와 같이 클래스를 독립적으로 분리한다면 DAO는 단순히 `int result = example.add();`와 같이 기능을 사용하면 된다. 

🚨
- 만약 ExampleClass의 메서드명이나 필드명을 변경한다면 우리는 해당 <span style="color:#F05750;">**변경사항을 일일히 수정**</span>해 주어야한다. 
- 어떤 클래스가 쓰일지, 해당 클래스의 메서드 이름이 무엇인지 **일일히 알고 있어야한다.** 


> 💭 다른 개발자가 내가 작성한 코드를 보거나 엄청나게 많은 기능을 포함한 코드를 몇년 후에 내가 본다고 생각해보자. **끔질할 것 같다...**

<br>


**3. 인터페이스**

인터페이스는 **어떤 일을 하겠다는 기능만 정의해 놓은 것**이다. 따라서, 구현 방법은 인터페이스의 **구현체에서 알아서 결정**하면 된다. 

위에서 정의한 ExampleClass 인터페이스의 구현체 A와 B 가 있다고 했을 때 

```java
public class AExampleClass implements ExampleClass{
    
    private String name = "Aexample";
    
    pulbic int add() {
        ... 
    }
    
    public String getName() {
        return this.name;
    }
}

public class BExampleClass implements ExampleClass{

    private String name = "Bexample";

    pulbic int add() {
        ...
    }

    public String getName() {
        return this.name;
    }
}
```
인터페이스를 통해 접근하기 때문에 **구체적인 클래스 [A, B 구현체]** 정보와 **메서드 정보**를 일일히 알지 않아도 된다. 

🔴 **중요한 점**
```java
public class Dao {
    private ExampleClass exampleClass;
    
    public Dao (Example example) {
        this.exampleClass = exampleClass; // 2-2 파트의 코드와 다르게 매개변수로 전달 받는다. 
    }
}
```

해당 코드와 같이 작성하여 Dao에서 구체적인 클래스 정보를 알 필요가 없어지게 된다.

위와 같이 구성한다면 `런타임 시점`에 코드에서는 <span style="color:orange;">**보이지 않던 관계가 오브젝트로 만들어진 후 생성**</span>된다. 

###  즉, 클라이언트는 Dao를 사용할 때 <ins>인터페이스의 구현 클래스를 선택</ins>하고 <ins>선택한 클래스의 오브젝트를 생성해서 DAO와 연결</ins>할 수 있다.

<br>

## 원칙과 패턴
### 개방 폐쇄 원칙 (OCP)
클래스나 모듈은 확장에는 열려 있어야 하고 변경에는 닫혀 있어야 한다. 

즉, 구현한 코드는 **변화에 영향을 받지 않고 유지하면서 확장도 되는 것**을 의미한다. 

>  💡 **인터페이스를 사용해 확장 기능을 정의한 대부분의 API는 개방 폐쇄원칙을 따른다.** 

<br>

> ### 객체지향 설계 원칙 (SOLID)
> 객체지향 기술은 점진적으로 발전해온 기술이라고 볼 수 있다. 앞서 살펴본 `디자인 패턴`은 <span style="color:orange;">**특별한 상황에서 발생하는 문제**</span>에 대한 솔루션이라고 한다면, `SOLID`는 좀 더 <span style="color:orange;">**일반적인 상황에서 적용 가능한 설계**</span> 기준이라고 볼 수 있다 

<br>

### 높은 응집도와 낮은 결합도 

`개방 폐쇄 원칙`은 **높은 응집도**와 **낮은 결합도**를 가진다. 

<br>

**[높은 응집도]**

- 응집도가 높다는 것은 하나의 모듈, 클래스가 <ins>**하나의 책임 or 관심사에만 집중**</ins>되어 있다는 의미이다.

  ➡︎ 패키지, 컴포넌트, 모듈에 이르기까지 그 대상의 크기가 달라도 동일하게 적용된다. 


<br>

**[낮은 결합도]**

- 높은 응집도보다 더 민감한 원칙으로 **느슨하게 연결된 형태를 유지하는 것**이 좋다. 
- 결합도가 낮아지면 <span style="color:orange;">**변화에 대응하는 속도가 높아지고 구성이 깔끔**</span>해진다. 

<br>

###  즉, <ins>한 클래스에 관심사가 집중적으로 몰려있어야 하며 각각의 클래스는 느슨한 연결</ins>이 되어 있어야 한다. 

<br>

## 전략 패턴 [전략(Strategy) 패턴 - 완벽 마스터하기](https://inpa.tistory.com/entry/GOF-%F0%9F%92%A0-%EC%A0%84%EB%9E%B5Strategy-%ED%8C%A8%ED%84%B4-%EC%A0%9C%EB%8C%80%EB%A1%9C-%EB%B0%B0%EC%9B%8C%EB%B3%B4%EC%9E%90)

`실행 (런타임) 중`에 알고리즘 전략을 선택하여 <span style="color:orange;">**객체 동작을 실시간으로 바뀌도록**</span> 할 수 있게 하는 행위 디자인 패턴이다.

**[예시 시나리오]**

온라인 쇼핑몰에서 물건을 구매하고 결제할 때, "카드", "현금", "계좌 이체" 중 하나를 선택해야 한다고 가정해보자. 
이때 전략 패턴을 활용하면 결제 방식을 유연하게 변경할 수 있다.  

```java
public interface Payment {
    String pay (int amount);
}
```
```java
public class CradPayment implements Payment{
    @Override
    String pay (int amount) {
        return amount + "원을 카드로 결제했습니다.";
    }
}
```
```java
public class CashPayment implements Payment{
    @Override
    String pay (int amount) {
        return amount + "원을 현금으로 결제했습니다.";
    }
}
```

<br>

## 제어의 역전(IOC)
`일반적인 프로그램의 흐름`은 프로그램이 **시작되는 지점에서 다음에 사용할 오브젝트를 결정** ➡ **︎결정한 오브젝트 생성** ➡︎ 만들어진 **오브젝트의 메서드 호출** ➡︎ 메소드 안에서 **다음에 사용할 것 결정 및 호출**

다만, 위와 같은 방식에서는 <ins>**모든 종류의 작업을 사용하는 쪽에서 제어하는 구조**</ins>이다. 

>💡 **제어의 역전이란?**
> 말 그대로 제어 흐름의 개념을 거꾸로 뒤집는 것을 의미한다. 
> <br> &nbsp; &nbsp; &nbsp;➡ 오브젝트는 <span style="color:#F05750;">**자신이 사용할 오브젝트를 스스로 선택 및 생성하지 않는다.**</span> 


🔴 **프로그램의 시작을 담당**하는 main()과 같은 `엔트리 포인트(애플리케이션 시작점)`는 제외


- 위의 예시 시나리오를 통해 가볍게 설명하자면, 구현체를 통해 만들어진 pay 메서드는 자신이 언제 호출될지 모르며 다른 오브젝트에 의해 필요할 때 실행된다.

<br>

### 라이브러리 VS 프레임워크

- 우리는 동작하는 중 **필요한 기능이 있을 때** 능동적으로 라이브러리를 사용한다. **[라이브러리]**
- `프레임워크가 흐름을 주도하는 중`에 <span style="color:orange;">**개발자가 만든 코드를 사용**</span>하도록 한다. **[프레임워크]**

###  IOC를 적용한다면 <ins>설계가 깔끔해지고 유연성이 증가하며 확장성이 좋아진다.</ins> 


<br>

## 스프링의 IOC
IOC는 `프레임워크`나 `컨테이너`와 같은 **애플리케이션 컴포넌트의 생성과 관계 설정, 사용, 생명 주기 관리 등을 관장하는 존재가 필요**하다. 

➡ ︎스프링에서는 **제어권을 가지고 직접 만들고 관계를 부여**하는 <span style="color:#E6B800;">**Bean**</span>이라는 오브젝트가 존재한다.

> 🟢 **생성과 제어를 담당하는 오브젝트만을 빈이라고 부른다.**

<br>

```java
@Configuration //애플리케이션 컨텍스트 or 빈 팩토리가 사용할 설정 정보라는 표시
public class DaoFactory {
    @Bean //오브젝트 생성을 담당하는 메소드라는 표시
    public Dao Dao() {
        ...
    }
}
```

<br>

> 💡 **빈 팩토리와 애플리케이션 컨텍스트** 
> <br>
> **`빈 팩토리:`** <span style="color:orange;">**핵심 컨테이너**</span>를 가르킨다. 빈 등록 및 생성, 조회, 부가적인 빈 관리 담당
> <br>
> <br>
> **`애플리케이션 컨텍스트:`** <span style="color:orange;">**빈 팩토리를 확장한 IOC 컨테이너**</span>이다. 기본적 기능은 빈 팩토리와 동일하고 각종 부가 서비스를 추가 제공한다. 

<br>

---

### 애플리케이션 컨텍스트의 동작 방식
**애플리케이션 컨텍스트**를 ➡︎ `IOC 컨테이너`, `스프링 컨테이너 (스프링)`라고 부르기도 한다. 

- **애플리케이션 컨텍스트**는 애플리케이션에서 **IOC를 적용해 관리할 모든 오브젝트에 대한 생성과 관계 설정을 담당**한다.


  ![Image](https://github.com/user-attachments/assets/939a4454-7051-4d00-8472-92e0a9f7a723)

- 클래스를 설정 정보로 등록 <span style="color:orange;">•• (1)</span> 
- @Bean붙은 메서드를 가져와 빈 목록을 만든다. <span style="color:orange;">•• (2)</span>
- 클라이언트가 호출 시 자신의 빈 목록에서 요청한 이름이 있는지 찾는다. <span style="color:orange;">•• (3)</span>
- 있다면 빈을 생성하는 메소드를 호출해서 오브젝트 생성 후 클라이언트에게 돌려준다. <span style="color:orange;">•• (4)</span> 

<br>

## 최종 정리 

- ### 클라이언트는 구체적인 팩토리 클래스를 알 필요가 없다. 

- ### 스프링 컨테이너는 종합 IOC 서비스를 제공해준다. 
    ➡ <span style="color:orange;">**︎오브젝트가 만들어지는 방식, 시점, 전략을 다르게 가져갈 수 있다!**</span>

- ### 스프링 컨테이너는 빈을 검색하는 다양한 방법을 제공한다. 