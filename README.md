# spring-template
* 
```
쉽게 스프링 프로젝트를 시작하기 위하기 위해 제작
Docker, docker-compose 활용하여 자동 빌드, 이미징, 배포, 실행
.bat 또는 .sh 로 간편하게 수행

Windows10 또는 Linux환경에서 개발 중에
스프링부트처럼 간단히
개발자가 자신의 데스크탑에 was를 설치할 필요없이
docker를 이용해 로컬에서 테스트를 위한 실행 가능하게함
```

* 커스텀화
```shell
# 이 레포지토리를 이용한 프로젝트 예상 구조
/projectRoot
    /spring-template-backend/**
    /react-frontend/**
    /someOtherProject/**
    docker-compose-dev.yml
    docker-compose-production.yml
    script.sh
    script.bat
    README.md
# 커다란 프로젝트 루트 폴더 밑에
# 지금 현재 레포지토리 spring-template위치하고
# react프론트엔드도 위치하고
# docker-compose작성하고
# script.sh 에 빌드, 이미징, 런 까지 포함해서 자동화
# !!! 필수 !!!
# spring의 config디렉토리, 등등 원하는 입맛에 맞게 재설정 필요
# docker-compose 입맛에 맞게 재설정 필요
```

* 포함
    - hibernate, hikariCP, h2(test), postgres(prod)
    - spring webmvc
    - spring security
    - lombok
    - index.jsp





# 실행 
* 필요사항
    - Openjdk8
    - Maven 
    - Docker
    - Docker-compose
    - WSL2(windows 10 환경일때)
* 실행 순서
```shell
mvn package -f .\ -Dmaven.test.skip=true
docker-compose -f docker-compose-dev.yml up -d 
docker-compose -f docker-compose-production.yml up -d

# 이것을 기반으로 스크립트파일 만들어서 통합해도됨
```




# Manual Maven Build & Docker
* Maven Build
    - mvn clean -f .\
    - mvn package -f .\
    - mvn package -f .\ -Dmaven.test.skip=true
* Dockfile
    - openjdk8 tomcat9 환경
    - docker build --tag spring-template:1.0 .
* 컨테이너 올리기
    - docker run --rm -p 80:8080 spring-template:1.0
* 도커컴포즈 올리기
    - 이름 docker-compose.yml일때 docker-compose up -d
    - 사용자정의 이름은 docker-compose -f something.yml up -d
    - rebuild 원할시 --build 옵션 추가 






# spring 키워드
* 빈 주입
    - Autowired, Qualifier
    - Inject, Named

---

* 값 유효성 검사
    - Validator 인터페이스
    - JSR380(빈 검증 2.0)
        ```java
        @NotNull, @Min, @Max, @NotBlank, @Size
        ```
    - 스프링의 JSR380지원 LocalValidatorFactoryBean
        ```java
        @Autowired private Validator validator;
        // org.springframework.validation.beanvalidation.LocalValidatorFactoryBean
        // jsr380의 validator, validatorFactory인터페이스 구현하는 동시에, 스프링 Validator인터페이스도 구현한다

        // 스프링 validator
        BeanPropertyBindingResult = bindingResult = new BeanPropertyBindingResult(대상, "Errors");
        validator.validate(대상, bindingResult);
        if(bindingResult.getErrorCount() > 0){
            logger.error("Error were found");
        }else{
            대상.작업
        }

        // jsr380 api
        Set<ConstraintViolation<대상.class>> violations = validator.validate(대상);
        Iterator<ConstraintViolation<대상.class>> iter = violations.iterator();
        if(itr.hasNext()) logger.error("Error were found");
        else 대상.작업
        ```
    - 메서드 검증
        ```java
        // 메서드의 인수와 반환값을 검증
        // org.springframework.validation.beanvalidation.MethodValidationPostProcessor

        //@Validated가 설정된 빈 클래스를 검색해 jsr380 제약 사항 애너테이션을 사용해 검증 지원한다

        @Validated
        public interface CustomerRequestService{
            @Future // 반환값은 미래날짜여야한다.
            Calendar submitRequest(@NotBlank String type, @Size(min=20, max=100) String description, @Past Calendar accountOpeningTime);
        }
        ```

--- 

* 프로파일
    - 빈 정의 프로파일
    - 빈 집합과 프로파일을 연결시켜서 환경에 따라 다른 빈을 사용하고싶을때 사용
    - ex) 개발환경에는 내장db, 프로덕션에서는 독립db
    - spring.profiles.active 프로퍼티 값으로 프로파일 이름을 설정
    - java 기반 config일때는 설정클래스또는 메소드에 @Profile({"dev", "~~"}) 등으로 프로파일 설정이 가능
    - 톰캣사용시 JVM 옵션을 통해 프로필 설정 가능 ex) -Dspring.profiles.active=dev 
    - 또는 web.xml에서 스위칭 ex) <context-param><param-name>spring.profiles.active</param-name><param-value>dev</param-value></context-param> 
        
--- 

* 스프링으로 데이터베이스 상호작용
    - 스프링은 JDBC 위에 추상계층을 추가해 데이터베이스와 상호작용을 편리하게 해줌
