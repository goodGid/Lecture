
## 2. 인터페이스의 변화

### 인터페이스 기본 메소드와 스태틱 메소드

> 200928 (Mon)

### 자바 8 API의 기본 메소드와 스태틱 메소드

> 200928 (Mon)


``` java
List<String> names = List.of("good", "gid", "goodgid");
Spliterator<String> spliterator = names.spliterator();
while (spliterator.tryAdvance(System.out::println));

// Output
good
gid
goodgid
```