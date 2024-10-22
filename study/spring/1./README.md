# 1. 오브젝트와 의존관계

## 서문

### 객체지향

* 스프링이 자바에서 가장 중요하게 가치를 두는 것은 바로 `객체지향` 프로그램이 가능한 언어라는 점이다.
* 자바 엔터프라이즈 기술 혼란 속 잃어버렸던 `객체지향`의 기술의 가치를 회복하고, `객체지향` 프로그래밍의 혜택을 누릴 수 있도록 **기본으로 돌아가자는 것**이 스프링의 핵심 철학이다.

### 오브젝트

* 스프링이 가장 관심을 많이 두는 대상은 `오브젝트`이다.
* 애플리케이션에서 오브젝트가 생성되고 생성되고 다른 오브젝트와 관계를 맺고, 사용되고, 소면되는 과정을 넘어 어떻게 설계되어야 하는지, 어떤 단위로 만들어지며 어떤 과정을 통배 등장해야 하는지에 대해 살펴봐야한다.

### 객체지향 설계

* 결국 이런 관심은 `객체지향 설계`의 기초와 원칙을 비롯해, `디자인 패턴`, `리팩토링`, `단위 테스트`와 같은 오브젝트 설계와 구현에 관한 여러 응용 기술과 지식이 요구된다.



* 스프링은 `객체지향` 설계와 구현에 관해 특정 모델과 기업을 강요하지 않는다.
* 스프링은`오브젝트`를 어떻게 효과적으로 설계하고, 구현하고, 개선해 나갈 수 있는 기준을 마련해준다
* 스프링은 `프레임워크`로서    객체지향 기술과 설계, 구현에 관한 실용적인 전략과 베스트 프랙티스를 자연스럽고 쉽게 적용할 수 있도록 제공한다.



## 초난감 DAO

{% code title="User Class" overflow="wrap" %}
```java
package springbook.user.admin;

public class User{
    String id;
    String name;
    String password;
    
    public String getId(){
        return id;
    }
    public String setId(String id){
        this.id = id;
    }
    
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    
    public String getPassword() {
        return password;
    }
    public void setPassword(String password){
        this.password = password;
    }
}
```
{% endcode %}



### main()을 이용한 DAO 테스트 코드

```java
public static void main(String[] args) throws ClassNotFoundException, SQLException {
    UserDao dao = new UserDao();
    
    User user = new User();
    uesr.setId("whiteship");
    user.setName("백기선");
    user.setPassword("married");
    
    dao.add(user);
    
    System.out.println(user.getId() + "등록 성공");
    
    User user2 = dao.get(user.getId());
    System.out.println(user2.getName());
    System.out.println(uesr2.getPassword());
    
    System.out.println(user2.getId() + " 조회 성공");
}
```





<details>

<summary>JDBC의 일반적인 작업 순서</summary>

1. DB 연결을 위한 Connection을 가져온다.
2. SQL을 담은 Statement를 만든다. (또는  PreparedStatement)
3. 만들어진 Statement 실행
4. 조회일 경우 SQL 쿼리의 실행 결과를 ResultSet으로 받아 정보를 저장할 오브젝트로 옮겨준다.
5. 작업 중 생성된 Connection, Statement, ResultSet같은 리소스는 작업 후에 닫아준다.
6. 예외 발생 - JDBC API가 생성하는 Exception을 잡아 직접 처리하거나, 메소드에 throws를 선언해 메소드 밖으로 던지게 한다.

</details>







