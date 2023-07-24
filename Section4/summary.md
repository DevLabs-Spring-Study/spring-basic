## 스프링 컨테이너와 스프링 빈

### 스프링 컨테이너 생성
- 스프링 컨테이너가 생성되는 과정
  - ApplicationContext (인터페이스 -> 다형성) 에 대해서, XML 기반 또는 애노테이션 기반 설정 클래스(AppConfig)로 만들 수 있음.
  - 컨테이너 : 객체를 담고 있는 것. BeanFactory와 ApplicationCOntext로 구분하느데, BeanFactory를 직접 사용하는 경우는 거의 x => ApplicationContext를 스프링 컨테이너로.
  - 스프링 컨테이너 생성하면서 AppConfig 정보를 넘겨줌 -> 스프링 컨테이너 생성, 그 안에는 빈 저장소가 있음. 빈 이름과 객체에 대한 저장소.
  - 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록함. @Bean 붙은 것들을 전부 호출해서 빈 저장소에 저장. method 이름을 빈 이름으로 설정하나, 임의로 부여할 수도 있음. 단, 항상 다른 이름으로 부여해야함.
  - 이후 빈 의존관계 설정을 준비함 -> 설정 정보를 참고해서 의존관계를 주입(DI) 함. 단순 자바 코드 호출과는 차이가 있음.
  - 스프링은 빈 생성과 의존관계 주입 단계가 나누어져 있음. 그런데 자바 코드로 빈을 등록하면 생성자 호출하면서 의존관계 주입도 한번에 처리하게 됨.

### 컨테이너에 등록된 모든 빈 조회
- 빈이 제대로 등록됐는지 조회를 해보자.
- ac.getBeanDefinitionNames() : 스프링에 등록된 모든 빈 이름을 조회.
- ac.getBean() : 빈 이름으로 빈 객체(인스턴스)를 조회.
- Role ROLE_APPLICATION : 직접 등록한 애플리케이션 빈
- Role ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈
  
### 스프링 빈 조회 - 기본
- ac.getBean(빈이름, 타입) 혹은 ac.getBean(타입) -> 만약 없으면 에러 발생
- 인스턴스로 조회할 수도 있음. 하지만 구체적으로 적는 것은 좋지 않음.
- 오류 발생 테스트코드 : assertThrows() 사용

### 스프링 빈 조회 - 동일한 타입이 둘 이상
- 타입 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류 -> 빈 이름을 지정해주면 된다.
- ac.getBeansOfType() 사용하면 해당 타입의 모든 빈 조회 가능

### 스프링 빈 조회 - 상속 관계
- 부모 타입으로 조회하면, 자식 타입도 함께 조회. 즉, 자바 객체의 최고 부모인 Object 타입으로 조회 -> 모든 스프링 빈 조회.
- 실제 개발에선 쓸 일이 거의 없지만, 가끔 순수 자바 어플리케이션에서 스프링 컨테이너를 생성해서 써야할 때 쓰게 됨. 특히 부모 타입 조회 -> 알고 있어야 자동 의존관계 주입에서 문제 없이 해결 가능.

### BeanFactory와 ApplicationContext
- BeanFactory <-(상속)- ApplicationContext <- AnnotationConfigApplicationContext(구현 객체)
- BeanFactory: 스프링 컨테이너 최상위 인터페이스. 빈을 관리하고 조회하는 역할 담당. getBean() 등등의 기능 제공.
- ApplicationContext: BeanFactory 기능 모두 상속받아서 제공. 어플리케이션 개발에는 빈 관리 조회뿐만 아니라 수많은 부가기능 필요 -> ApplicationContext는 BeanFactory외에도 그 외 다양한 인터페이스들도 상속받음.
  - 메시지소스를 활용한 국제화 기능(한국어권->한국어, 영어권->영어), 환경변수(로컬, 개발, 운영 등을 구분해서 처리 : 환경별 DB 등...), 어플리케이션 이벤트(이벤트 발행, 구독 모델 편리하게 지원), 편리한 리소스 조회(파일, 클래스패스, 외부 등에서 리소스 편리하게 조회)
- 즉, ApplicationContext는 빈 관리기능+편리한 부가기능. BeanFactory나 ApplicationContext를 스프링 컨테이너라고 함.

### 다양한 설정 형식 지원 - 자바 코드, XML
- 스프링 컨테이너는 다양한 형식 설정 정보를 받아들일 수 있도록 유연하게 설계됨.
  - AnnotationConfigApplicationContext : AppConfig.class
  - GenericXmlApplicationContext : appConfig.xml
  - XxxApplicationContext : appConfig.xxx
- 최근에는 XML 사용 잘 하지 않음. 하지만 레거시 프로젝트 중 XML 사용하는 것들이 많고, 컴파일 없이 빈 설정 정보를 변경할 수 있는 장점도 있음.
- 자바 코드로 된 AppConfig.java와 비교해보면 거의 비슷함.

### 스프링 빈 설정 메타 정보 - BeanDefinition
- 어떻게 다양한 설정형식을 지웧날까? -> **BeanDefinition**이라는 추상화. **역할과 구현을 개념적으로 나눈것**
  - XML, 자바 코드, ... 를 읽어서 BeanDefinition을 만들면 됨.
  - BeanDefinition : 빈 설정 메타 정보 => @Bean, <bean> 당 각각 하나씩 메타 정보가 생성되는 것.
  - 즉, 스프링 컨테이너는 이 메타정보를 기반으로 스프링 빈을 생성한다.
- 각각의 ApplicationContext 객체는 BeanDefinitionReader가 있다 : 파일을 읽어서 BeanDefinition 설정 정보를 만들어냄.
- BeanDefinition 정보
  - BeanClassName : 생성할 빈의 클래스 명 (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)
  - factoryBeanName : 팩토리 역할의 빈을 사용할 경우 이름
  - factoryMethodName : 빈을 생성할 팩토리 메서드 지정
  - Scope : 싱글톤 (기본값)
  - lazyInit : 스프링 컨테이너를 생성할때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때까지 최대한 생성을 지연처리 하는지 여부
  - initMehodName : 빈을 생성하고, 의존 관계를 적용한 뒤에 호출되는 초기화 메서드 명
  - DestroyMethodName : 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
  - Constructor arguments, Properties : 의존관계 주입에서 사용 (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음)
  - 이러한 메타정보를 기반으로 객체를 생성하는 것임.
- 직접 스프링 빈 등록 or 팩토리 메서드를 통해서 등록(외부에서 메서드 호출해서 생성되는 방식:일반적인 자바 방식 등록)으로 크게 두 갈래로 나뉨.
- BeanDefinition을 직접 생성해서 스프링 컨테이너에 등록할 수도 있다. 가끔 오픈소스 등을 볼 때 보일 때가 있으니, 알고만 있으면 됨.
- cf) 팩토리 메서드 패턴 : 부모 클래스에서 객체들을 생성할 수 있는 인터페이스를 제공하지만, 자식 클래스들이 생성될 객체들의 유형을 변경할 수 있도록 하는 생성 패턴. [참고](https://refactoring.guru/ko/design-patterns/factory-method)
