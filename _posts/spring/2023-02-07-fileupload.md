---
published: true
title : "SpringBoot - 파일업로드"
categories:
  - Spring

tags:
  - Spring
  - JPA

toc: true
toc_sticky: true
sidebar:
  nav: "sidebar-category"
---

# 1. 들어가며
  &nbsp; 지난 프로젝트들을 통해서 여러 게시판을 만들어 봤지만, 파일업로드 기능을 구현해본적이 없는것 같아서 정리겸 한번 만들어보았다.
<br><br>

## 준비
&nbsp; 기본적으로 SpringBoot의 gradle로 프로젝트를 만들고, JPA와 RDB 하나를 사용합니다. 이 글에서는 MariaDB를 사용합니다. 기본 셋팅까지는 여러 블로그나 글에도 설명이 많이 되어 있기 때문에 생략하고 진행하겠습니다.
<br>
<br>

기본적인 개요는 업로드된 이미지나 파일을 DB에 올리는것이 아닌, 서버에 저장하도록 만들고, 파일의 경로만 DB에 저장을 시킵니다.

<br>
<br>


먼저 파일 업로드를 위해 Board Entitiy와 Repository를 만들어줍니다.

```
@NoArgsConstructor
@Getter
@Setter
@Entity
public class Board {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  private String title;
  private String content;
  private String filename;
  private String filepath;
}

public interface BookRepository extends JpaRepository<Book, Long>{}

```

<br>
이제 파일업로드를 위한 BookService를 만들어봅시다.

```
@Service
@RequiredArgsConstructor
public class BoardService {

  private final BoardRepository boardRepository;

  public void write(Board board, MultipartFile file) throws Exception {
    //MultipartFile 인터페이스는 스프링에서 업로드 한 파일을 표현할 때 사용되는 인터페이스입니다.


    String projectPath =
      System.getProperty("user.dir") + "\\src\\main\\resources\\static\\files";
    
    //식별자 = 랜덤으로 이름을 만듭니다.
    UUID uuid = UUID.randomUUID();

    // 같은 파일 이름이 올라가면 덮어쓰기가 됨로, 랜덤식별자를 앞에 붙여 저장될 파일의 이름을 구분합니다.
    String fileName = uuid + "_" + file.getOriginalFilename();

    //projectPath 위치에 fileName으로 파일을 생성
    File saveFile = new File(projectPath, fileName);

    //받아온 객체를 업로드 처리하지 않으면 임시파일에 저장된 파일이 자동적으로 삭제되기 때문에 MultipartFile객체의 transferTo(File f) 메서드를 이용해서 업로드처리를 해야 한다.
    file.transferTo(saveFile);


    board.setFilename(fileName);
    board.setFilepath(
      "/files/" + fileName
    );

    boardRepository.save(board);
  }

```

<br>

여기까지 왔으면 백엔드 부분의 FileUpload 기능의 구현은 완료된것 입니다.
Controller 단에서 받아주기만 하면 됩니다.

```
 @PostMapping("board/writepro")
  public void boardWritePro(Board board, MultipartFile file)
    throws Exception {
    boardService.write(board, file);
  }

  이런식으로 말입니다..
  만약 템플릿엔진을 사용하고 있다면, Model을 사용하셔서 해당 화면으로 적절한 값을 리턴해주시면 됩니다.
```

<br>
<br>

혹시 필요하신분이 있을까봐 업로드단의 html파일도 간략히 만들어 봤습니다

```
<body>
    <div class="layout">
    <!--form에서 enctype="multipart/form-data"-->
      <form
        action="/board/writepro"
        method="post"
        enctype="multipart/form-data"
      >
        <input name="title" type="text" />
        <textarea name="content"></textarea>
        <!--파일 선택추가-->
        <input type="file" name="file" />
        <!--name이름을 controller의 매개변수 이름과 동일하게 설정-->
        <button type="submit">작성</button>
      </form>
    </div>
  </body>
```


이상으로 간단히 만들어본 파일업로드 시스템 이었습니다. 해당 업로드된 파일을 사용하실땐 DB에 있는 filepath을 가져와서 사용하시면 되겠습니다.













