---
title : "[FreezerProject] JSON -> CSV 후처리"
categories:
  - Project

tags:
  - 개발
  - 자바
---

   # 들어가며

   &nbsp; 첫 글부터 갑자기 프로젝트 이야기지만, 뒤 늦게 블로그를 시작한만큼 차차 공부한 내용과 프로젝트 내용들을 정리해서 올릴날을 기대해본다. <br>&nbsp;깃블로그는 걸음마 단계라 글 가독성이 떨어지는점 다시 한번 사과드립니다.

<br>
  
   # 프로젝트 소개

   &nbsp; 이번 프로젝트는 기본적으로 `SpringBoot`와 `React`를 활용한 서버와 프론트단이 완전히 분리되어 있는 Restful 서비스로서 `MariaDB` 서버를 채택하여 제작중에 있다.<br>
    &nbsp; 로그인을 통해 가상의 냉장고를 만들어 재료를 담을수 있으며, 냉장고에 넣어 둔 재료를 기반으로 만들수 있는 메뉴 소개 및 만드는 법을 알려주는 정도의 간단한 웹어플리케이션이다.<br>

  # 오늘의 문제
  &nbsp; `JPA`를 활용해 더미데이터만을 사용해서 테스트만 해봤지, 크롤링으로 수집한 대용량 `JSON`파일들을 파싱해서 DB에 넣는 방법에 대해서 고민해본적이 없었다. 

   최근에는 `MYSQL`이나 `MariaDB` 자체적으로 `JSON`타입을 지정해서 DB에 바로 담을수 있다고는 하지만, 그렇게 넣을시엔 정확하게 원하는 값을 가져오는데 문제가 있기 때문에, 고민하다가 포기한 방법중 하나이다. JSON 자체를 DB에 담으면 JSON 한 행이 통채로 넘어 오기 때문에 정확하게 파싱해서 원하는 값만 받기가 어렵다고 한다.

   어쩔수 없이 JSON파일을 원하는 형태의 CSV 파일로 전처리 한후 DB에 강제로 집어 넣는 방식으로 결정하고, 오랜 삽질 끝에 DB에 무사히 자료들을 넣을수 있었다.

   파이썬은 오랜만에 다뤄보는거라 많이 서툴어서 한참을 애먹고서야 원하는 데이터프레임으로 가공할수 있었다는 후문..
<br>
<br>

   ```
import pandas as pd
import warnings
import sys

warnings.filterwarnings('ignore')

data_file=pd.read_csv('recipe.csv',encoding='cp949')

df_melted=pd.melt(data_file,id_vars=['menu_title'],value_vars=['ingrs/0/ingr_name','ingrs/1/ingr_name','ingrs/2/ingr_name','ingrs/3/ingr_name','ingrs/4/ingr_name','ingrs/5/ingr_name','ingrs/6/ingr_name','ingrs/7/ingr_name'])

#print(df_melted)
df_melted.to_csv("레시피재료.csv",index=False)
  ```
<br>
<br>
  내가 원하는 키 값과 밸류 값을 원하는 방향으로 재구성하는 코드 이다.<br> 녹여서 바꾼다고 melt라는 문법 이라고 하네요.<br>
  <br>
  <br>


# 마무리
  공부 할것도 많고, 프로젝트 할것도 많고, 시간은 모자라고, <br>잠도 모자라는 개발자 꿈나무는 오늘도 밤늦게 이러고 있었답니다..



   


   
