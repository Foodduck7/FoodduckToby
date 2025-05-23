# Chapter6 스프링의 AOP

<br>

## 스프링의 프록시 팩토리 빈

스프링은 `프록시 오브젝트를 생성`해주는 **기술을 추상화한 팩토리 빈을 제공**해준다. 

### ProxyFactoryBean

- 프록시를 생성해서 빈 오브젝트로 등록해주는 역할을 한다.
- 순수하게 프록시를 **생성하는 작업만 담당**하고 프록시를 통해 제공해줄 `부가기능은 별도의 빈`에 둘 수 있다. 

프록시 팩토리 빈에서 생성하는 **프록시에 사용할 부가 기능의 경우** `MethodeInterceptor 인터페이스를 구현`해서 만든다.

<br>

> ### 💡 충고하는 사람(Advisor)은 누구(Pointcut)에게 어떤 충고(Advice)를 해야 할지 알고있다

<br>

#### 🔎 어드바이스 (타깃이 필요 없는 순수한 부가기능)

`타깃 오브젝트(실체)`에 종속되지 않는 <ins>**순수한 부가기능을 담은 오브젝트이다.**</ins> 


#### 🔎 포인트컷 (부가기능 적용 대상 메소드 선정 방법)

메소드 선정 알고리즘이다. 

#### 🔎 포인트컷 (포인트컷 + 어드바이스)

**어드바이스와 포인트컷을 묶은 오브젝트**를 어드바이저라고 부른다. 

> ### **💭 여러개의 어드바이스와 포인트컷이 추가된다면?**
> **어떤 어드바이스**에 **어떤 포인트컷**을 적용할지 애매해진다. 
> 
> ➡︎ 조합하여 <span style="color:orange;">**Advisor 타입으로 만들어 등록**</span>한다. **(여러가지로 조합 가능)**


<br>

**[전체적인 흐름]**

![Image](https://github.com/user-attachments/assets/03484f6e-54e6-4c5d-af0d-b208492aa3cc)

1. `프록시`는 클라이언트로부터 요청을 받으면 먼저 <ins>**포인트컷에게 부가 기능을 부여할 메소드인지 확인**</ins>해달라고 요청
2. 포인트 컷으로부터 확인을 받으면, <ins>**MethodInteceptor 타입의 어드바이스를 호출**</ins>한다. 
3. `어드바이스`는 프록시로부터 전달받은 MethodInvocation 타입 <ins>**콜백 오브젝트의 proceed()메소드를 호출**</ins>한다. 

<br>

## 스프링의 AOP 

### DefaultAdvisorAutoProxyCreator

어드바이저를 이용한 **자동 프록시 생성기**이다. (스프링이 제공하는 빈 후처리기)

<br>

**[적용 방법]**

- **빈 후처리기를 스프링 빈으로 등록**한다.
- 스프링이 `빈을 생성할 때마다` 빈 후처리에 **작업을 요청**한다.
- 빈 후처리기는 빈을 **수정**하거나, **초기화**하거나, 아예 **다른 객체(프록시)로 바꿔치기**할 수도 있다.

>  💡빈 후처리기를 활용하면 **스프링이 생성한 빈**을 <ins>**프록시로 감싸서 등록**</ins>할 수 있다. 


<br>


### 💭 복잡하고 세밀한 기준을 이용해 클래스나 메소드를 선정하려면 어떻게 해야할까? 
### 포인트컷 표현식 (AspectJ 포인트컷 표현식)

- `AspectExpressionPointcut 클래스`를 사용하여 **클래스와 메소드의 포인트컷을 한번에 지정**한다. 
- AspectJ라는 유명한 프레임워크에서 제공하는 것을 가져와 일부 붐법을 확장새서 사용한다. 

<br>

####  🔎 execution(): 주로 지시자를 사용한 포인트컷 표현식 사용


![Image](https://github.com/user-attachments/assets/52b546d8-a01f-46cb-baed-3496b2516be5)


>  💡 `execution(* 메서드명(..))`
> 
> **'*' (와일드카드)** 사용 시 타입 무시
> 
> **'..'** 사용시 파라미터 개수와 타입을 무시한다.
 

<br>

### AOP란 무엇인가? ➡︎ 애스팩트 지향 프로그래밍/ 관점 지향 프로그래밍

**[전체적인 과정]: 500 ~ 504page 다시 읽어보기** 

Aspect란 애플리케이션의 핵심기능을 담고 있지는 않지만 **애플리케이션을 구성하는 중요한 한가지 요소**이고 핵심 기능에 부가되어 **의미를 갖는 특별한 모듈을 가르킨다.** 

- OOP(객체 지향 프로그래밍)를 돕는 보조적인 기술이다. 
- AOP는 `공통 기능`을 **따로 분리해서 관리하는 방법**이다.

<br>

### AOP 적용 기술 

**스프링 AOP는 `프록시 방식의 AOP`이다.** 


### 💭 그렇다면 프록시 방식이 아닌 AOP에는 무엇이 있을까?  ➡︎ AspectJ

> **💡 AspectJ는** 
>
> ➡︎ 컴파일된 타깃의 클래스 파일 자체를 수정하거나
> 
> ➡︎ 클래스가 JVM에 로딩되는 시점을 가로채서 바이트 코드를 조작하여 부가 기능을 직접 넣어주는
> 
> ➡︎ 직접적인 방식을 사용한다. 
> 
> **💭 왜 사용할까?**
> 
> - **스프링과 같은 컨테이너가 사용되지 않는 환경에서도 AOP 적용 가능** 
> - 프록시보다 훨씬 강력하고 유연한 AOP가 가능


<br>

### AOP 용어 

- **조인 포인트(Join Point)**

  ➡︎
  어드바이스가 적용될 수 있는 위치를 의미한다.

  ➡︎
  타깃 오브젝트가 구현한 모든 메소드는 조인 포인트가 된다.

<br>


### AOP 네임스페이스

**[스프링의 프록시 방식 AOP를 적용하기 위한 4가지 빈]** 

- 자동 프록시 생성기
- 어드바이스
- 포인트 컷
- 어드바이저
