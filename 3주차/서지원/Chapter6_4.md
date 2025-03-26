# Chapter6 어노테이션 트랜잭션 속성과 포인트 컷

<br>

## 트랜잭션 어노테이션

### @Transactional 

![Image](https://github.com/user-attachments/assets/d7ea5641-824e-474c-88ce-7e48f0e493e0)

![Image](https://github.com/user-attachments/assets/64f24a52-3358-4a8f-b8de-21840ef6aef4)

**스프링**은 @Transactional이 부여된 <ins>**모든 오브젝트를 자동으로 타깃 오브젝트로 인식**</ins>한다. 

이때, 사용되는 포인트 컷은 `TransactionAttributeSourcePointcut`이다. 

➡️ @Transactional은 기본적으로 **트랜잭션 속성을 정의**하는 것이지만 동시에 **포인트컷의 자동등록에도 사용**된다.  

<br>

#### 대체 정책 

**🗨️ @Transactional을 반복적으로 매소드마다 부여하게 된다면?**

- 코드가 지저분해 진다. 
- 동일한 속성을 가진 어노테이션이 중복된다.

<br>

스프링에서는 이러한 문제를 `4단계의 대체 정책 (fallback)`을 이용하여 해결한다. 

**메소드의 속성을 확인할 때** `타깃 메소드 ➡️ 타깃 클래스 ➡️ 선언 메소드 ➡️ 선언 타입` 의 순서에 따라 <ins>**적용 여부를 확인**</ins>하고 가장 <ins>**먼저 발견되는 속성정보를 사용**</ins>한다. 

![Image](https://github.com/user-attachments/assets/cd05b239-9ddb-4a2d-936b-54f0917f351f)


<br>

> **💡 테스트에도 @Transactional 적용이 가능하다.** 
> 
> 다만, **테스트 코드에서 실행이 정상적으로 끝나면** @Transactional은 강제 `롤백`을 시키기 때문에 <ins>**DB에 작업이 반영되지 않는다.**</ins>
> 
> ➡️ 반영을 원한다면 @Rollback 을 사용한다. 


> 💡 `@Rollback`은 **메소드 레벨에서만 가능**하기 때문에 <ins>전체적으로 롤백의 속성 부여를 원한다면 @TransactionalConfiguration을 사용</ins>한다. 


<br>

### 선언적 트랜잭션과 트랜잭션 전파 속성

#### 선언적 트랜잭션
AOP를 이용해 코드 외부에서 트랜잭션의 기능을 부여해주고 <ins>**속성을 지정할 수 있게 하는 방법**</ins>

**ex> @Transactional** 

<br>

#### 프로그램에 의한 트랜잭션
TransactionTemplate OR 개별 데이터 기술의 트랜잭션 API를 사용해 코드 안에서 사용하는 방법

**ex> JDBC, EntityManager와 EntityTransaction를 이용한 트랜잭션**