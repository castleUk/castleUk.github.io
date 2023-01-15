---
published: true
title : "Int와 Integer 차이가 뭘까?"
categories:
  - Java

tags:
  - Spring
  - JPA
---

# 1. 들어가며
<br>
<center><img src="https://static.javatpoint.com/core/images/int-vs-integer-java.png " width="100%" height="100%"></center>
<br>

  &nbsp; 오늘도 의식의 흐름에 따라 코딩을 하던 중
  `Entity` 설계를 하던 중 count를 표시하기 위해 무의식적으로 *int* 타입으로 작성을 하고 있었다.  다른 레퍼런스를 보던 중 다른 분들의 `Entitiy`에는 *Integer type*으로 선언 되어 있는것을 발견! 음.... 둘의 차이가 뭐지?
  단순히 숫자와 객체의 차이인건가? 기본타입과 참조타입? 너무 기초적이지만 입이 안떨어지는 물음에 내가 이렇게 기초적인 것도 대답못하면서 코딩을 공부하고 있었다는 생각에 충격을 먹었다고 한다..
  <br><br>


# 2. int vs Integer 뭐가 다르냐?

`결론부터 보고 가자!`

  > int는 기본형(primitive type) 데이터타입<br>
  > Integer는 int를 객체로 다루기 위해 사용되는 Wrapper Class이다.

<br>


| | int      | Integer                  |
|--------| --------- | --------------- |
|1|   primitive 자료형 | Wrapper클래스(객체)                   |
|2| 산술연산이 가능하다.   | 산술연산 불가능 |
|3| null값 사용 불가 | null값 처리 가능|

<br>

## 2-1 primitive type vs reference 
<br>

### <center>`primitive type(기본형) 이란?`<br>
boolean, char, byte, short, int, long, float, double 실제 연산에 사용될수 있는 기본 변수 타입<br><br>

### <center> `reference type(참조형) 이란?`<br>
기본형 8가지를 제외한 나머지 타입을 통칭, 참조형 변수는 변수 타입으로 클래스의 이름을 사용, 클래스의 이름이 참조형 변수의 타입이 됨

## 2-2 Wrapper Classes
&nbsp;기본 타입의 데이터를 객체로 표현해야 하는 경우 primitive type을 객체를 다루기 위해 사용하는 클래스를 wrapper class라고 함
<br><br>
<center><img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FpVd25%2FbtrGhVKXykm%2FLjGml14W0APWmtqVQoPfz1%2Fimg.png " width="80%" height="80%"></center>
<br>



# 3. 예제

`제네릭 안에서는 반드시 Wrapper class`
<br>
```
ArrayList<Integer> Arr = new ArrayList<>();  // 올바른 형식
    ArrayList<int> Arr2 = new ArrayList<>();  // 잘못된 형식
```

`null값 처리 할때는 반드시 Wrapper class`<br>
```
      	Integer a = null;  // 올바른 형식
      	int b = null;  // 잘못된 형식
```

`JPA Entity 필드값 처리시 Integer 사용`

```
private int count;
private Integer count;
```
사실 둘중 무엇을 사용해도 되나 Hibernate JPA 공식문서에 따르면 WrapperClass 를 추천하고 있다. 
<br>
<br>

# 정리
- int는 기본형, Integer는 int의 Wrapper Class로 객체를 나타냄
- 산술연산 가능유무의 차이
- null값 처리 유무의 차이
- 제네릭 안 JPA에서는 Wrapper Class!










