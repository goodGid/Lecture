

## 5. Date와 Time

### 5.1 Date와 Time 소개

> 200930 (Wed)

* Java 8에서 Date와 Time API가 나온 이유는

  이전까지는 개판이여서 그렇다.

* [Joda Time API](https://www.joda.org/joda-time/)를 사용했는데

  이게 스펙으로 인정되어서 자바 8에 추가되었다.

* 주요 API
    
    - 기계용 시간과 인간 친화적 시간 구분 가능

    - 기간을 표현할 때 Duration(시간 기반)과 Period(날짜 기반) 사용 가능


---


### 5.2 Date와 Time API

> 200930 (Wed)

* 기계용 시간

  **Instant**를 사용한다.

``` java
Instant instant = Instant.now();
// 기준시 (UTC == GMT)
System.out.println(instant); // 2020-09-29T22:57:35.775207Z

ZoneId zone = ZoneId.systemDefault();
System.out.println(zone); // Asia/Seoul

ZonedDateTime zonedDateTime = instant.atZone(zone);
System.out.println(zonedDateTime); // 2020-09-30T07:57:35.775207+09:00[Asia/Seoul]
```

* 인간 친화적 시간

``` java
/**
* 한국에서 개발하고 미국에서 서버를 배포했다면
* LocalDateTime은 미국 시간을 기준이 된다.
*/
LocalDateTime now = LocalDateTime.now();
System.out.println(now); // 2020-09-30T08:01:56.545393

/**
* 원하는 시간 만들기
*/
LocalDateTime customDate = LocalDateTime.of(2020, Month.AUGUST, 16,0, 0, 0, 0);
System.out.println(customDate); // 2020-08-16T00:00

/**
* 보고싶은 Zone Time 보기
*/
ZonedDateTime nowInUSA = ZonedDateTime.now(ZoneId.of("America/Los_Angeles"));
System.out.println(nowInUSA); // 2020-09-29T16:04:26.833273-07:00[America/Los_Angeles]
```

---

* Period 

  인간을 위한 시간을 비교

``` java
LocalDate today = LocalDate.now();
System.out.println("today : " + today);

LocalDate endDay = LocalDate.of(2020, Month.DECEMBER, 31);
System.out.println("endDay : " + endDay);

Period period = Period.between(today, endDay);
System.out.println("Month difference : " + period.getMonths());

Period until = today.until(endDay);
System.out.println("Day difference : " + until.get(ChronoUnit.DAYS));

## Output
today : 2020-09-30
endDay : 2020-12-31
Month difference : 3
Day difference : 1
```

---

* Duration

  기계를 위한 시간 비교

* **Instant**를 갖고 비교한다.


``` java
Instant now = Instant.now();
System.out.println("today : " + now);

Instant plusSeconds = now.plus(10, ChronoUnit.SECONDS);
System.out.println("plus seconds : " + plusSeconds);

Duration between = Duration.between(now, plusSeconds);
System.out.println("Seconds difference : " + between.getSeconds());

## Output
today : 2020-09-29T23:17:58.843279Z
plus seconds : 2020-09-29T23:18:08.843279Z
Seconds difference : 10
```

---


* DateTimeFormatter

``` java
LocalDateTime now = LocalDateTime.now();
System.out.println(now); // 2020-09-30T08:20:35.663296

DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MM/dd/yyyy");
System.out.println(now.format(formatter)); // 09/30/2020
```

* Custom하게 DateTimeFormatter를 만들 수 있고

  미리 정의된 [Formatter](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/time/format/DateTimeFormatter.html#predefined)를 사용할 수 있다.


---

* LocalDate.parse

``` java
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("MM/dd/yyyy");
LocalDate parse = LocalDate.parse("08/16/1992", formatter);
System.out.println(parse); // 1992-08-16
```


---

* Date <-> Instant 

``` java
Date date = new Date();
System.out.println(date);

Instant instant = date.toInstant();
System.out.println(instant);

Date newDate = Date.from(instant);
System.out.println(newDate);

## Output
Wed Sep 30 08:32:53 KST 2020
2020-09-29T23:32:53.872Z
Wed Sep 30 08:32:53 KST 2020
```