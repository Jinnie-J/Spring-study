
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
  

### 싱글톤 컨테이너
#### 스프링 없는 순수한 DI 컨테이너
- AppConfig는 요청할 때 마다 객체를 새로 생성한다.
- 고객 트래픽이 초당 100이 나오면 초당 100개 객체가 생성되고 소멸된다. -> 메모리 낭비가 심하다.
- 해결방안은 해당 객체가 딱 1개만 생성되고, 공유하도록 설계하면 된다. -> 싱글톤 패턴
  
#### 싱글톤 패턴
- 클래스의 인스턴스가 딱 1개만 생성되는 것을 보장하는 디자인 패턴이다.
- 그래서 객체 인스턴스 2개 이상 생성하지 못하도록 막아야 한다.
- private 생성자를 사용해서 외부에서 임의로 new 키워드를 사용하지 못하도록 막아야 한다.
- 싱글톤 패턴을 적용하면 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 사용할 수 있다.
  
#### 싱글톤 패턴 문제점
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존관계상 클라이언트가 구체 클래스에 의존한다. -> DIP를 위반한다.
- 클라이언트가 구체 클래스에 의존해서 OCP 원칙을 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화 하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
  
#### 싱글톤 컨테이너
- 스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하면서, 객체 인스턴스를 싱글톤(1개만 생성)으로 관리한다.
- 지금까지 만든 스프링 빈이 바로 싱글톤으로 관리되는 빈이다.
- 스프링 컨테이너는 싱글톤 컨테이너 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라 한다.
- 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있다.
  - 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.
  - DIP, OCP, 테스트, private 생성자로 부터 자유롭게 싱글톤을 사용할 수 있다.
  
#### 싱글톤 방식의 문제점
- 싱글톤 패턴이든, 스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스르 공유하기 때문에 싱글톤 객체는 상태유지(stateful) 하게 설계하면 안된다.
- 무상태(stateless)로 설계해야 한다.
  - 특정 클라이언트에 의존적인 필드가 있으면 안된다.
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.
  - 가급적 읽기만 가능해야 한다.
  - 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, ThreadLocal 등을 사용해야 한다.
- 스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다. 

#### @Configuration과 바이트코드 조작
- 스프링 컨테이너는 싱글톤 레지스트리다. 따라서 스프링 빈이 싱글톤이 되도록 보장해주어야 한다. 그러기 위해 스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용한다.
- @Bean이 붙은 메서드마다 이미 스프링 빈이 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.
- @Configuration을 적용하지 않고, @Bean만 적용할 경우 - 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않는다.
- 스프링 설정 정보는 항상 @Configuration을 사용하자.


### 컴포넌트 스캔

#### 컴포너트 스캔과 의존관계 자동 주입
- 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
- 컴포넌트 스캔은 이름 그대로 @Component 애노테이션이 붙은 클래스를 스캔해서 스프링 빈으로 등록한다.
- @Configuration이 컴포넌트 스캔의 대상이 된 이유도 @Configuration 소스코드를 열어보면 @Component 애노테이션이 붙어있기 때문이다.
- @Autowired를 사용하면 생성자에서 여러 의존관계도 한번에 주입받을 수 있다.

#### 탐색 위치와 기본 스캔 대상
- 모든 자바 클래스를 다 컴포넌트 스캔하면 시간이 오래 걸린다. 그래서 꼭 필요한 위치부터 탐색하도록 시작 위치를 지정할 수 있다.
- basePackages: 탐색할 패키지의 시작 위치를 지정한다. 이 패키지를 포함해서 하위 패키지를 모두 탐색한다.
- basePackageClasses: 지정한 클래스의 패키지를 탐색 시작 위로 지정한다.
- 만약 지정하지 않으면, @ComponentScan이 붙은 설정 정보 클래스의 패키지가 시작 위치가 된다.
- 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 두는 것이 좋다. 최근 스프링 부트도 이 방법을 기본으로 제공한다.

- 컴포넌트 스캔 기본 대상
  - @Component: 컴포넌트 스캔에서 사용
  - @Controller: 스프링 MVC 컨트롤러에서 사용
  - @Service: 스프링 비즈니스 로직에서 사용.
  - @Repository: 스프링 데이터 접근 계층에서 사용. 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다.
  - @Configuration: 스프링 설정 정보에서 사용. 스프링 빈이 싱글톤을 유지하도록 추가 처리를 한다.

#### 필터
- includeFilters: 컴포넌트 스캔 대상을 추가로 지정한다.
- excludeFilters: 컴포넌트 스캔에서 제외할 대상을 지정한다.
- FilterType 옵션
  - ANNOTATION: 기본값, 애노테이션을 인식해서 동작한다. ex) org.example.SomeAnnotation
  - ASSIGNABLE_TYPE: 지정한 타입과 자식 타입을 인식해서 동작한다. ex) org.example.SomeClass
  - ASPECTJ: AspectJ 패턴 사용 ex) org.example..*Service+
  - REGEX: 정규표현식 ex) org\.example\.Default.*
  - CUSTOM: TypeFilter 이라는 인터페이스를 구현해서 처리 ex) org.example.MyTypeFilter
- 참고: @Component면 충분하기 때문에, includeFilters를 사용할 일은 거의 없다. excludeFilters는 여러가지 이유로 간혹 사용할 때가 있지만 많지는 않다. 최근 스프링 부트는 컴포넌트 스캔을 기본으로 제공하므로, 옵션을 변경하면서 사용하기 보다는 스프링의 기본 설정에 최대한 맞추어 사용하는 것을 권장한다.
  
### 의존관계 자동 주입
#### 다양한 의존관계 주입 방법
- 의존관계 주입은 크게 4가지 방법이 있다.
  - 생성자 주입
  - 수정자 주입(setter 주입)
  - 필드 주입
  - 일반 메서드 주입

- 생성자 주입  
  - 이름 그대로 생성자를 통해서 의존 관계를 주입 받는 방법이다.
  - 특징
    - 생성자 호출시점에 딱 1번만 호출되는 것이 보장된다.
    - '불변,필수' 의존관계에 사용
    - 생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입 된다. (물론 스프링 빈에만 해당된다.)

- 수정자 주입(setter 주입)
  - setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방식이다.
  - 특징
    - '선택,변경' 가능성이 있는 의존관계에 사용
    - 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.
  
- 필드 주입
  - 필드에 바로 주입하는 방식이다.
  - 특징
    - 코드가 간결하지만 외부에서 변경이 불가능해서 테스트 하기 힘들다는 치명적인 단점이 있다.
    - DI 프레임워크가 없으면 아무것도 할 수 없다.
    - 사용하지 말자 (애플리케이션의 실제 코드와 관계 없는 테스트코드에서만 사용)

- 일반 메서드 주입
  -일반 메서드를 통해서 주입 받을 수 있다.
  - 특징
    - 한번에 여러 필드를 주입 받을 수 있다.
    - 일반적으로 잘 사용하지 않는다.