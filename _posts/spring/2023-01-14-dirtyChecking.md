---
published: true
title : "JPA 더티 체킹(Dirty Checking)"
categories:
  - Spring

tags:
  - Spring
  - JPA
---

# 1. 들어가며
  &nbsp; 진행중인 프로젝트에 JPA를 사용하고 있다. 새로운 기술의 적용은 항상 설레고 두근거리는 일이지만, 제대로 이해하지 못하고 사용할바엔 사용하지 않느니만 못하기에 오늘도 열심히 공부한 내용을 기록한다.
<br><br>

# 2. Entity 생명주기
<center><img src="https://velog.velcdn.com/images%2Fneptunes032%2Fpost%2Fecd3b113-862f-4158-a208-e1eeec92d61d%2Fimage.png " width="80%" height="80%"></center>
<br>

- 영속성 컨텍스트 : Entity를 영구 저장하는 환경
- 비영속(new/transient): 영속성 컨텍스트와 전혀 관계가 없는 상태
- 영속(managed): 영속성 컨텍스트에 저장된 상태
- 준영속(detached): 영속성 컨텍스트에 저장되었다가 분리된 상태
- 삭제(removed): 삭제된 상태
<br>
<br>

# 3. 더티체킹(Dirty Checking) 이란?

- JPA는 Entity Manager가 Entity를 CRUD 한다.
- Entity Manager 내부 메소드를 살펴 보면 저장(persist)/조회(find)/삭제(delete)로 수정에 해당하는 메서드가 없음.
- 대신 더티 체킹을 통해 수정을 하도록 함
- 더티 체킹은 Transaction 안에서 엔티티의 변경이 일어나면, 변경 내용을 자동으로 데이터베이스에 반영하는 JPA 특징입니다.
- 영속성 컨텍스트 안의 엔티티를 대상으로 일어남

## `더티체킹이 일어나는 시점은?`
= EntityManager flush
  >영속성 컨텍스트의 변경 내용을 DB에 반영하는 메소드
- entitiyManager.flush() 직접 호출
- 트랜잭션 커밋 시점에 자동으로 호출
- JPQL 쿼리 실행시, 실행 전에 자동으로 호출

# 4. 코드예제
  &nbsp; 실제 코드로 살펴보자.

  ```
@Service
public class ExampleService {
     @Transactional
  public void update(Long id, String name) {
      Optional<User> user = userRepository.findById(id);
      if(user.isPresent()) {
          User userInfo =user.get();
          userInfo.setName(name)
}
  ```

User Entity의 이름을 변경하는 setName 메소드 만으로도 트랜잭션이 걸려 있으면, 트랜잭션이 끝나는 순간 더티 체킹을 통해 이름이 업데이트가 된다. `repository.save()를 따로 안해줘도 된다.` 
> 트랜잭션을 사용하지 않는다면 반드시 save 메소드를 날려 줘야 업데이트 가능!

# 5. 정리
- 트랜잭션이 걸려 있으면, 영속 엔티티의 변경사항이 트랜잭션이 끝나는 순간 자동으로 반영됨
- save 메서드를 따로 안써줘도 된다.
- JPA는 공부할게 많다.
- 난 오늘도 살아 있다.













