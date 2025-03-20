# Chapter6 트랜잭션 속성

### 트랜잭션 매니저에서 트랜잭션을 가져올 때 사용하는 DefaultTransactionDefinition 

![Image](https://github.com/user-attachments/assets/b585964c-e7c6-49df-91ea-cf5f658e5823)

DefaultTransactionDefinition이 구현하고 있는 `TransactionDefinition 인터페이스`는 트랜잭션의 동작 방식에 영향을 줄 수 있는 <ins>**4가지 속성을 정의하고 있다.**</ins> 

### 트랜잭션 전파 

위의 4가지 속성을 확인하기 전에 `트랜잭션 전파(Transaction propagation)`에 대해 알아보자

트랜잭션 전파란 **이미 진행 중인 트랜잭션이 있을 때** <ins>**새로운 트랜잭션을 어떻게 처리할지 결정하는 방식이다.**</ins> 

<br>

### [참여하는 경우]
![Image](https://github.com/user-attachments/assets/62314906-ada8-4443-9c02-9ba174993a6e)


### [새로운 트랜잭션 생성]
![Image](https://github.com/user-attachments/assets/5412d19a-e474-4f28-bc1f-76231ee19c56)

<br>

### 1️⃣ PROPAGATION_REQUIRED

- 가장 많이 사용되는 트랜잭션 전파 속성이다. 
- 진행 중인 트랜잭션이 없다면 새로 시작하고, 이미 시작된 트랜잭션이 있다면 이에 참가한다. 

<br>

### 2️⃣ PROPAGATION_REQUIRED_NEW

- 항상 새로운 트랜잭션을 시작한다.
- 독자적으로 동작하게 한다. 
- 독립적인 트랜잭션이 보장돼야 하는 코드에 적용 가능 

<br>

### 3️⃣ PROPAGATION_NOT_SUPPORTED

- 트랜잭션 없이 동작하도록 만들 수 있다. 
- 진행 중인 트랜잭션이 있어도 무시한다. 


<br>

### 격리 수준 (Isolation Level)  ➡️ ISOLATION_DEFAULT

![Image](https://github.com/user-attachments/assets/7a571b47-facd-4f74-b8e1-331c5f6b06e4)

**서버 환경에서는 여러 개의 트랜잭션이 동시에 진행될 수 있다.** 

가능하다면 순차적으로 진행되는 것이 좋겠지만 이는 **성능이 크게 떨어지고** **사용자에게 좋지 않은 경험을 남길 수 있다.** 

따라서 적절히 `격리 수준을 조정`하여 <ins>**가능한 한 많은 트랜잭션을 동시에 진행 시켜도 문제가 발생하지 않도록 제어할 필요가 있다.** </ins>

<br>

### 제한 시간

트랜잭션을 수행하는 제한 시간 

**기본적인 설정은 제한시간이 없는 것**이다. 

<br>

### 읽기 전용 ➡️ read only

읽기 전용으로 설정해두면 트랜잭션 내에서 데이터를 조작하는 시도를 막아줄 수 있다. 

또한 엑세스 기술에 따라 성능이 향상될 수 있다. 

> ### 💬 GetMapping의 경우 read-only를 했을 때 성능이 향상된다고 했었는데 정확한 이유를 모르겠다. 


<br>

## 트랜잭션 인터셉터와 트랜잭션 속성 

### 💬 이미 **DB에 쓰기 작업이 진행된 채**로 **읽기전용 트랜잭션 속성을 가진 작업이 진행**된다면 `충돌`될까? **NO**

**readOnly나 timeout 등**은 <ins>**트랜잭션이 처음 시작될 때가 아니라면 적용되지 않는다.**</ins> 

<br>

### 프록시 방식 AOP는 타깃 오브젝트 내의 메소드를 호출할 때는 적용하지 않는다. 

위의 문장에 대한 예시는 아래와 같다.

![Image](https://github.com/user-attachments/assets/2af0e5c4-fbd0-4b53-9de4-b2f7ae3a8afc)


사진의 `[1]`, `[3]`과 같이 트랜잭션 **프록시를 통해 타깃 메소드로 전달**된다면 <ins>**트랜잭션 경계설정 부가기능이 부여된다.**</ins> 

**But**, `[2]`처럼 프록시를 거치지 않고 **직접 타깃의 메소드를 호출**한다면 <ins>**트랜잭션 속성이 적용되지 않는다.**</ins> 



**[예시]**

```java
@Service
public class OrderService {
    
    @Transactional
    public void placeOrder() {
        System.out.println("상품 주문 시작");
        saveOrder();  // 트랜잭션 적용 안 됨 
    }

    @Transactional
    public void saveOrder() {
        System.out.println("주문 정보 저장");
    }
}
```

> 💡 같은 클래스 내의 메소드를 호출하게되면 Spring 프록시 과정을 거치지 않기 때문에 적용되지 않는다. 