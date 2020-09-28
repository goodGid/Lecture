

## 3. Stream

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

<br>

* 병렬 처리(Parallel)가 가능하다.

  무조건적으로 좋은건 아니다. 

* Why? 

  Context Switching 등등

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