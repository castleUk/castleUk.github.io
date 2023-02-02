---
published: true
title : "exception rollback(Checked, Unchecked Exception)의 잘못된 이해"
categories:
  - Java

tags:
  - Java

sidebar:
    nav: "sidebar-category"
---

## 정리
- 트랜잭션이란것은 예외처리에 대한 규칙이 없다
- 트랜잭션은 의미가 많음. 어떤 트랜잭션인지 말해야함
- DB트랜잭션의 경우 checked, unchecked Exception에서 rollback을 할꺼냐 말꺼냐는 우리가 정하는거임. 규칙이 없음
- Checked Exception이라고 예외발생시 롤백하지 않고 Unchecked Exception이라고 롤백해야 한다는 틀린 표현이다.
- Spring의 트랜잭션 처리에서는 기본적으로 Runtime계열은 바로 rollback을 하고 unchecked 같은 경우 rollback 하지 않는다. 하지만 설정할수도 있음.
- Java의 트랜잭션의 처리를 Spring 트랜잭션이라고 동일시 하지 말기








