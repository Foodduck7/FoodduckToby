# Chapter4 예외 

<br>

## ⚠️예외 처리 시 하면 안 되는 코드
### 로그나 메세지에 금방 묻혀 놓치기 쉬운 코드이다.

```java
try {

} catch (Exception e) {
        System.out.println(e); 
}
```


<br>

### 모든 예외는 무조건 던져버리는 무책임한 throws 선언도 심각한 문제이다.

#### ➡︎ `적절한 처리를 통해 복구될 수 있는 예외상황`도 제대로 다룰 수 있는 <span style="color:#F05750;">기회를 박탈 당한다.</span>

```java
public void method1() throws Exception {
    
}

public void method2() throws Exception {

}

public void method3() throws Exception {

}
```



<br>

## 🚀 예외 처리 시 반드시 지켜야하는 핵심 원칙 한가지

### <span style="color:#F05750;">예외</span>는 <ins>**적절하게 복구**</ins>되거나 <ins>**작업을 중단시키고 운영자 또는 개발자에게 분명하게 통보**</ins>돼야 한다.


<br>

## 예외의 종류와 특징

**🔵 Error**

- <span style="color:orange;">**시스템에 뭔가 비정상적인 상황이 발생했을 경우 사용**</span>된다.  
- 주로 자바 **VM에서 발생시키는 것으로 코드에서 잡으려고 하면 안된다.** 

<br>

**🔵 Exception & 체크 예외**

- 개발자들이 만든 <span style="color:#F05750;">**애플리케이션 코드의 작업 중 예외 상황이 발생**</span>했을 경우 사용한다.

    <br>

  - **체크 예외**
  
    Exception 클래스의 서브 클래스이면서 `RunTimeException을 상속하지 않는 것`
  
  <br>
  
  - **언체크 예외**
  
    `RunTimeException을 상속한 것` 
    
    
> **💡 RunTimeException은 Exception의 서브 클래스이다.** 



<br>

**🔵 RunTimeException과 언체크/런타임 예외**

대표적인 RunTimeException 예외 ➡︎ `NullPointerException`, `IlliegalArgumentException`


<br>

## 예외 처리 방법 

### 예외 복구 

**예외 상황을 파악하고 문제를 해결해서 정상 상태로 돌려놓는 것**을 말한다. 

**ex>** 네트워크가 불안해서 가끔 서버 접속이 잘 안되 SQLException이 발생하는 경우 예외 상황으로 부터 복구를 시도할 수 있다.  


<br>

### 예외 처리 회피 

**예외처리를 자신이 담당하지 않고 자신을 호출한 쪽으로 던져버리는 것을 말한다.** 

- `Throws문`으로 선언해서 <span style="color:orange;">**예외가 발생하면 알아서 던진다.** </span>
- `catch문`으로 <span style="color:orange;">**예외를 잡고 로그를 남긴 후 다시 예외를 던진다.**</span> 

**[예외처리 회피1]**

```java
public void method() throws Exception {
    
}
```


**[예외처리 회피2]**
```java
public void method() throws Exception {

    try {

    } catch (Exception e) {
        throw e;
    }
}
```


<br>

### 예외 전환 

예외를 메소드 밖으로 던지는 것이다. 

`발생한 예외`를 그대로 넘기는게 아니라 **적절한 예외로 전환해서 던지는 특징**이 있다. 


<br>


**[목적1]**

<span style="color:orange;">**의미를 분명하게 해줄 수 있는 예외로 바꿔주기 위해서**</span> 사용한다.

➡︎ 의미가 분명한 예외가 던져지면 **서비스 계층 오브젝트에는 적절한 복구 잡업을 시도할 수 있게된다.** 


> **💡 원래 발생한 예외를 담아서 `중첩 예외`로 만드는 것이 좋다.** 
> 
>  ➡︎  중첩 예외는 `getCause() 메서드`를 이용해서 **처음 발생한 예외가 무엇인지 확인할 수 있다.** 
> 
>  ➡    생성자나 initCause() 메소드로 근본 원인이 되는 예외를 넣어주면 된다. 

<br>


**[중첩 예외1]**

```java
catch(Exception e) {
    throw DulicateUserIdException(e);    
}
```

**[중첩 예외2]**

```java
catch(Exception e) {
    throw DulicateUserIdException().initCause(e);    
}
```


<br>


**[목적2]**

예외를 처리하기 쉽고 단순하게 만들기 위해 <span style="color:orange;">**포장(wrap)하는 것**</span> 

➡︎ 위와 방법은 동일하지만 `의미를 명확하게 하려고 다른 예외로 전환하는 것은 아니다.`

> 💡 주로 예외처리를 강제하는 `체크 예외`를 언체크 예외인 `런타임 예외`로 **바꾸는 경우 사용한다.** 


<br>

> **💡 `EJBException: ` RuntimeException을 상속하고 있다.** 
> 
> EJB는 런타임 예외를 Exception으로 인식하여 트랜잭션을 자동으로 롤백시켜준다. 



### 📌 `어쩌피 복구하지 못할 예외`라면 애플리케이션 <ins>코드에서는 런타임 예외로 포장해서 던져버리고</ins>, 예외처리 서비스 등을 이용해 `개발자`에게는 <span style="color:orange;">해당 예외를 알려주고</span> `사용자`에게는 <span style="color:orange;">안내 메세지를 보여주는 식</span>으로 처리하는 것이 좋다. 


<br>

### 애플리케이션 예외

시스템 또는 외부의 예외 상황이 원인이 아니라 애플리케이션 자체의 로직에 의해 <ins>**의도적으로 발생 시키는 것을 의미**</ins>한다. 

<br>

**[처리 방법1]**

<span style="color:orange;">**결과 상태를 나타내는 정보를 return 값으로 활용**</span>한다. **ex>  0, -1 등의 특별한 값 리턴**

➡︎  if문 코드가 많아지고 코드가 지저분해질 수 있다.


<br>

**[처리 방법2]**

예외 상황 발생 시 비즈니스적인 의미를 띈 예외를 던지도록 만드는 것이다. 


<br>

### SQLException

<span style="color:#F05750;">**JDBC의 SQLException은 대부분 복구할 수 없는 예외**</span>이기 때문에 이를 **런타임 예외로 변환하여 다루어야 한다.** 

또한, **SQLException의 에러 코드는 데이터베이스에 따라 다르기 때문에** 특정 <span style="color:#F05750;">**DB에 종속된 예외 처리를 하면 DB 변경 시 코드 수정이 필요해지는 문제가 존재**</span>한다. 

스프링은 이러한 문제를 해결하기 위해 `DataAccessException 계층을 제공`하여 데이터 접근 예외를 **추상화된 런타임 예외로 변환**한다.  

이를 통해 특정 DB에 종속되지 않고 예외를 일관되게 처리할 수 있다.


