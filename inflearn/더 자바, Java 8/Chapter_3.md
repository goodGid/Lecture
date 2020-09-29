

## 3. Stream

### 3.1 Stream 소개 

> 200928 (Mon)

* 스트림은 기존의 데이터 소스를 변경하지 않는다.

``` java
List<String> names = List.of("good", "gid", "goodgid");

// 스트림 정의만 했기 때문에 실행 X
names.stream().map((s) -> s.toUpperCase());

// 종료 오퍼레이션을 사용했기 때문에 실행이 된다.
List<String> collect = names.stream().map((s) -> {
    System.out.println("Origin Value : " + s);
    return s.toUpperCase();
}).collect(Collectors.toList());

System.out.println("==========");

// 기존 데이터를 UpperCase로 변경
collect.forEach(System.out::println);

System.out.println("==========");

// 기존 데이터는 그대로 유지
names.forEach(System.out::println);

// Output
Origin Value : good
Origin Value : gid
Origin Value : goodgid
==========
GOOD
GID
GOODGID
==========
good
gid
goodgid
```

---

* 병렬 처리(Parallel)가 가능하다.

  무조건적으로 좋은건 아니다. 

* Why? 

  Context Switching 등 문제가 있다.

  그래서 병렬 처리는 데이터가 엄청나게 많을 경우에 좋다.

``` java
List<String> names = List.of("good", "gid", "goodgid");

// 병렬 처리 O
// 각각 다른 Thread에서 작업 진행
System.out.println("[Case 1]");
List<String> collect1 = names.parallelStream().map(s -> {
    System.out.println("Value : " + s + " | Thread Name : " + Thread.currentThread().getName());
    return s;
}).collect(Collectors.toList());

collect1.forEach(System.out::println);

System.out.println("==========");

// 병렬 처리 X
// 동일한 Thread에서 작업 진행
System.out.println("[Case 2]");
List<String> collect2 = names.stream().map(s -> {
    System.out.println("Value : " + s + " | Thread Name : " + Thread.currentThread().getName());
    return s;
}).collect(Collectors.toList());

collect2.forEach(System.out::println);

// Output
[Case 1]
Value : gid | Thread Name : main
Value : good | Thread Name : ForkJoinPool.commonPool-worker-5
Value : goodgid | Thread Name : ForkJoinPool.commonPool-worker-3
good
gid
goodgid
==========
[Case 2]
Value : good | Thread Name : main
Value : gid | Thread Name : main
Value : goodgid | Thread Name : main
good
gid
goodgid
```

* Case 1에서 **Thread Name** 값은 

  프로그램을 실행시킬 때 마다 달라진다.


  ---


  ### 3.2 Stream API

  > 200930 (Wed)

  * 실습하는 강의였어서 

    굳이 다시 들을 필욘 없다.

``` java
List<OnlineClass> springClasses = new ArrayList<>();
springClasses.add(new OnlineClass(1, "spring boot", true));
springClasses.add(new OnlineClass(2, "spring data jpa", true));
springClasses.add(new OnlineClass(3, "spring mvc", false));
springClasses.add(new OnlineClass(4, "spring core", false));
springClasses.add(new OnlineClass(5, "rest api development", false));

System.out.println("spring 으로 시작하는 수업");
Collection<OnlineClass> collect1 =
        springClasses.stream()
                      .filter(oc -> oc.getTitle().startsWith("spring"))
                      .collect(Collectors.toList());
collect1.forEach(i -> System.out.println(i.getId()));
System.out.println();

System.out.println("close 되지 않은 수업 = false 값들 찾기");
springClasses.stream()
//                     .filter(Predicate.not(OnlineClass::isClosed)) // Method Reference 사용
              .filter(oc -> oc.isClosed() ? false : true)
              .forEach(oc -> System.out.println(oc.getId()));
System.out.println();

System.out.println("수업 이름만 모아서 스트림 만들기");
List<String> collect2 = springClasses.stream()
//                                             .map(OnlineClass::getTitle) // Method Reference 사용
                                      .map(oc -> oc.getTitle())
                                      .collect(Collectors.toList());
collect2.forEach(oc -> System.out.println(oc));
System.out.println();

List<OnlineClass> javaClasses = new ArrayList<>();
javaClasses.add(new OnlineClass(6, "The Java, Test", true));
javaClasses.add(new OnlineClass(7, "The Java, Code manipulation", true));
javaClasses.add(new OnlineClass(8, "The Java, 8 to 11", false));

List<List<OnlineClass>> totalClasses = new ArrayList<>();
totalClasses.add(springClasses);
totalClasses.add(javaClasses);

System.out.println("두 수업 목록에 들어있는 모든 수업 아이디 출력");
totalClasses.stream()
            .flatMap(i -> i.stream())
            .forEach(i -> System.out.println(i.getId()));
System.out.println();

System.out.println("10부터 1씩 증가하는 무제한 스트림 중에서 앞에 10개 빼고 최대 10개 까지만");
List<Integer> integerList = Stream.iterate(10, i -> i + 1)
                                  .skip(10)
                                  .limit(10).collect(Collectors.toList());
integerList.forEach(i -> System.out.println(i));

System.out.println();

System.out.println("자바 수업 중에 Test가 들어있는 수업이 있는지 확인");

// 의도를 잘못 파악해서 짠 코드
javaClasses.stream()
            .filter(i -> i.getTitle().contains("Test"))
            .forEach(i -> System.out.println(i.getTitle()));

// 의도를 제대로 파악한 코이게 정상적인 코드
boolean test = javaClasses.stream()
                          .anyMatch(i -> i.getTitle().contains("Test"));
System.out.println("자바 수업 중에 Test가 들어가 있는가? : " + test);
System.out.println();

System.out.println("스프링 수업 중에 제목에 spring이 들어간 것만 모아서 List로 만들기");
List<OnlineClass> spring = springClasses.stream()
                                        .filter(i -> i.getTitle().contains("spring"))
                                        .collect(Collectors.toList());
spring.forEach(i -> System.out.println(i.getTitle()));

## Output
spring 으로 시작하는 수업
1
2
3
4

close 되지 않은 수업 = false 값들 찾기
3
4
5

수업 이름만 모아서 스트림 만들기
spring boot
spring data jpa
spring mvc
spring core
rest api development

두 수업 목록에 들어있는 모든 수업 아이디 출력
1
2
3
4
5
6
7
8

10부터 1씩 증가하는 무제한 스트림 중에서 앞에 10개 빼고 최대 10개 까지만
20
21
22
23
24
25
26
27
28
29

자바 수업 중에 Test가 들어있는 수업이 있는지 확인
The Java, Test
자바 수업 중에 Test가 들어가 있는가? : true

스프링 수업 중에 제목에 spring이 들어간 것만 모아서 List로 만들기
spring boot
spring data jpa
spring mvc
spring core
```

* 직접 모든 질문에 대해 답을 맞췄다.