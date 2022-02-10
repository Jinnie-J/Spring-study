
# 스프링 핵심원리 - 기본편

### 스프링 핵심 원리 이해 - 회원, 주문, 할인  도메인
- 역할과 구현을 분리해서 자유롭게 구현 객체를 조립할 수 있게 설계 -> 회원 저장소, 할인 정책도 유연하게 변경 가능

  ![spring](https://user-images.githubusercontent.com/62706198/152681880-1e9b2a02-ee54-4fc8-b500-f8fdd2109fb0.PNG)


### 스프링 핵심 원리 이해 - 객체 지향 원리 적용

#### 문제점: 할인 정책을 변경하려면 클라이언트의 코드를 고쳐야 한다.
```
public class OrderServiceImpl implements OrderService{
//private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
```
- 추상(인터페이스) 뿐만 아니라 구체(구현) 클레스에도 의존하고 있다.
- 위의 코드는 기능을 확장해서 변경하면, 클라이언트의 코드에 영향을 준다. -> OCP를 위반한다.
- DIP를 위반하지 않도록 인터페이스에만 의존하도록 의존관계를 변경하면 된다.


#### 인터페이스에만 의존하도록 코드 변경
```
  Public class OrderServiceImpl implements OrderService{
  //private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
  private Discount discountPolicy;
```
- 인터페이스에만 의존하도록 설계와 코드를 변경했지만, 구현체가 존재하지 않아서 NPE(null pointer exception)가 발생한다.
- 해결방안: 누군가가 클라이언트인 OrderServiceImpl에 DiscountPolicy의 구현 객체를 대신 생성하고 주입해주어야 한다.


#### 구체 클래스가 아닌 인터페이스에만 의존하도록 설계
- 관심사의 분리
  - 애플리케이션을 하나의 공연이라고 생각해보자. 각각의 인터페이스를 배역이라 생각하자.
  - 배우는 본인의 역할인 배역을 수행하는 것에만 집중해야 한다.
  - 디카프리오는 어떤 여자주인공이 선택되더라도 똑같이 공연을 할 수 있어야 한다.
  - 공연을 구성하고, 담당 배우를 섭외하고, 역할에 맞는 배우를 지정하는 책임을 담당하는 별도의 공연기획자가 나올시점이다.
  - 공연 기획자를 만들고, 배우와 공연 기획자의 책임을 확실히 분리하자.

#### AppConfig 등장
- 애플리케이션의 전체 동작 방식을 구성(config)하기 위해, 구현 객체를 생성하고, 연결하는 책임을 가지는 별도의 설정 클래스를 만든다.
- AppConfig는 애플리케이션의 실제 동작에 필요한 '구현 객체를 생성'한다.
- AppConfig는 생성한 객체 인스턴스의 참조(레퍼런스)를 '생성자를 통해서 주입(연결)' 해준다.
  ![appconfig](https://user-images.githubusercontent.com/62706198/152795713-95efabc7-cecc-4af7-98a0-c27330cffefd.PNG)
- 객체의 생성과 연결은 AppConfig가 담당한다.
- DIP 완성 - MemberServiceImple은 MemberRepository인 추상에만 의존하면 된다. 이제 구체 클래스를 몰라도 된다.
- appConfig 객체는 memoryMemberRepository 객체를 생성하고 그 참조값을 memberServiceImpl을 생성하면서 생성자로 전달한다.
- 클라이언트인 memberServiceImpl 입장에서 보면 의존관계를 마치 외부에서 주입해주는 것 같다고 해서 DI 우리말로 의존관계 주입 또는 의존성 주입이라 한다.
- Appconfig를 생성한 후, 할인 정책을 변경하려면 한 곳만 변경해주면 된다. "사용 영역"의 어떤 코드도 변경할 필요가 없다.  
  ```
  public DiscountPolicy discountPolicy(){
    //return new FixDoscountPolicy();
    return new RateDiscountPolicy(); 
  ```
#### IOC, DI, 그리고 컨테이너
- 제어의 역전 IoC(Inversion of Control)
  - 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다. 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다.
  - 반면에 AppConfig가 등장한 이후에 구현 객체는 자신의 로직을 실행하는 역할만 담당한다. 프로그램의 제어 흐름은 AppConfig가 가져간다. 예들 들어서 OrderServiceImpl은 필요한 인터페이스들을 호출하지만 어떤 구현 객체들이 실행될지 모른다.
  - 이렇듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC)이라 한다.

- 의존관계 주입 DI(Dependency Injection)
  - 의존관계는 "정적인 클래스 의존관계와, 실행 시점에 결정되는 동적인 객체(인스턴스) 의존 관계" 둘을 분리해서 생각해야 한다.
  - 정적인 클래스 의존관계
    - 클래스가 사용하는 import 코드만 보고 의존관계를 쉽게 판단할 수 있다. 정적인 의존관계는 애플리케이션을 실행하지 않아도 분석할 수 있다.
    - 그런데 이러한 클래스 의존관계 만으로는 실제 어떤 객체가 주입 될지 알 수 없다.
  - 동적인 객체 인스턴스 의존관계
    - 애플리케이션 실행 시점에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계다.
    - 애플리케이션 실행시점(런타임)에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해서 클라이언트와 서버의 실제 의존관계가 연결되는 것을 "의존관계 주입"이라 한다.
    - 의존관계 주입을 사용하면 정적인 클래스 의존관계를 변경하지 않고, 동적인 객체 인스턴스 의존관계를 쉽게 변경할 수 있다.

- Ioc 컨테이너, DI 컨테이너
  - AppConfig 처럼 객체를 생성하고 관리하면서 의존관계를 연결해 주는 것
  - 의존관계 주입에 초점을 맞추어 최근에는 주로 DI 컨테이너라 한다.
  - 또는 어셈블러, 오브젝트 팩토리 등으로 불리기도 한다.

#### 스프링으로 전환하기
- 스프링 컨테이너
  - "ApplicationContext"를 스프링 컨테이너라 한다.
  - 기존에는 개발자가 'AppConfig'를 사용해서 직접 객체를 생성하고 DI를 했지만, 이제부터는 스프링 컨테이너를 통해 사용한다.
  - 스프링 컨테이너는 '@Configuration'이 붙은 'AppConfig'를 설정(구성) 정보로 사용한다. 여기서 '@Bean'이라 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다. 이렇게 스프링 컨테이너에 등록된 객체를 스프링 빈이라 한다.
  - 기존에는 개발자가 직접 자바코드로 모든 것을 했다면 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 먼테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.

### 스프링 컨테이너와 스프링 빈

- 스프링 컨테이너가 생성되는 과정
  ```
  ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
  ```
- ApplicationContext를 스프링 컨테이너라 한다. ApplicationContext는 인터페이스이다.
- 스프링 컨테이너는 XML을 기반으로 만들 수 있고, 애노테이션 기반의 자바 설정 클래스로 만들 수 있다.
- 위에서 AppConfig를 사용했던 방식이 애노테이션 기반의 자바 설정 클래스로 스프링 컨테이너를 만든 것이다.
- 스프링 컨테이너의 생성 과정
  1. 스프링 컨테이너 생성   
     ![spring1](https://user-images.githubusercontent.com/62706198/153409808-9d591a52-ca47-4c6a-98e2-dd99a52bd27e.JPG)
  2. 스프링 빈 등록   
     ![spring2](https://user-images.githubusercontent.com/62706198/153218328-0cd801ea-12cc-4fb9-9f08-bb5c9c06e9f6.PNG)
  3. 스프링 빈 의존관계 설정 - 준비   
     ![spring3](https://user-images.githubusercontent.com/62706198/153218376-614c6f38-00fb-475f-b3c3-97c76c1f2dce.PNG)
  4. 스프링 빈 의존관계 설정 - 완료   
     ![spring4](https://user-images.githubusercontent.com/62706198/153218435-e958b517-577a-4507-86ca-9ea91d62c9ce.PNG)
     

#### BeanFactory와 ApplicationContext
- BeanFactory
  - 스프링 컨테이너의 최상위 인터페이스다.
  - 스프링 빈을 관리하고 조회하는 역할을 담당한다.
  - getBean()을 제공한다.
- ApplicationContext
  - contextFactory 기능을 모두 상속받아서 제공한다.
  - 빈을 관리하고 검색하는 기능을 BeanFactory가 제공해준다.
  - 메시지소스를 활용한 국제화 기능: 한국에서 들어오면 한국어로, 영어권에서 들어오면 영어로 출력
  - 환경변수: 로컬, 개발, 운영등을 구분해서 처리
  - 애플리케이션 이벤트: 이벤트를 발행하고 구독하는 모델을 편리하게 지원
  - 편리한 리소스 조회: 파일, 클래스패스, 외부 등에서 리소스를 편리하게 조회
  
#### 다양한 설정 형식 지원 - 자바코드, XML
- 스프링 컨테이너는 다양한 형식의 정보를 받아드릴 수 있게 유연하게 설계되어 있다. (자바코드, XML, Groovy 등등)
- 애노테이션 기반 자바 코드 설정 사용
  - 'new AnnotationConfigApplicationContext(AppConfig.class)'
  - 'AnnotationConfigApplicationContext' 클래스를 사용하면서 자바 코드로된 설정 정보를 넘기면 된다.
- XML 설정 사용
  - 최근에는 스프링 부트를 많이 사용하면서 XML 기반의 설정은 잘 사용하지 않는다. 아직 많은 레거시 프로젝트 들이 XML로 되어있고, 또 XML을 사용하면 컴파일 없이 빈 설정 정보를 변경할 수 있는 장점이 있다.
  - 'GenericXmlApplicationContext'를 사용하면서 'xml'설정 파일을 넘기면 된다.