---
layout: post
title: Spring MVC와 MyBatis, Bootstrap으로 기본적인 게시판 구현하기!
---

# Spring MVC로 게시판 만들기

안녕하세요?
이번에 진행할 간단한 프로젝트는 스프링 MVC 레거시 프로젝트로 게시판을 만들어보기입니다.

일단, 사용할 스펙들은 다음과 같습니다.

* Design: Bootstrap
* IDE: Spring Tool Suite
* Backend: Spring MVC
* Database: MyBatis

딱히 뭐 별거없죠?
이제 시작해볼까요?

# 프로젝트 생성

STS에 들어가셔서 Spring MVC Legacy Project로 프로젝트 하나를 생성해주세요!

# pom.xml 세팅

자 이제, 생성이 되었으니, 가장중요한 pom.xml로 갑니다!
일단, java버전을 저는 1.8로 사용할건데요, 자바 9 으로 하셔도 관계없습니다!

```xml
<properties>
<java-version>1.8</java-version>
<org.springframework-version>4.3.8.RELEASE</org.springframework-version>
<org.aspectj-version>1.6.10</org.aspectj-version>
<org.slf4j-version>1.6.6</org.slf4j-version>
</properties>
```

저는 이런식으로 기본세팅을 맞추었습니다.
바뀐부분은, java-version과 springframework-version입니다.

그리고, 이제 라이브러리들을 추가해볼까요?

```xml
<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis -->
<dependency>
<groupId>org.mybatis</groupId>
<artifactId>mybatis</artifactId>
<version>3.4.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.mybatis/mybatis-spring -->
<dependency>
<groupId>org.mybatis</groupId>
<artifactId>mybatis-spring</artifactId>
<version>1.3.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-test -->
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-test</artifactId>
<version>${org.springframework-version}</version>
<scope>test</scope>
</dependency>

<!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>5.1.38</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.springframework/spring-jdbc -->
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-jdbc</artifactId>
<version>${org.springframework-version}</version>
</dependency>
```

자, 필요 라이브러리들은 위와 같습니다! 일단 spring-jdbc및 spring-test를 추가해주시고, 버전을 스프링프레임웍 버전과 동일하게 맞춰주시면 되고, mybatis연결을위해 나머지들을 추가해주세요!

# root-context.xml

이제 루트 컨텍스트를 건드려볼까요?

일단 namespaces에 aop, beans, context, jdbc, mybatis-spring을 추가해주시고,

다음과 같이 datasource및 sqlsessiontemplate, sqlsessionfactory를 추가해줍시다!
그리고 서버 실행시 자동으로 스캔할수 있도록 component-scan태그도 달아줍시다!

package들에 대해 설명 하자면,
persistance패키지에는 @Repository가 붙는 DAO파일이 들어갑니다.
그다음 service 패키지에는 @Service 어노테이션이 붙는 서비스 파일이 들어갑니다.

```xml
<bean id="dataSource" class="org.springframework.jdbc.datasource.DriverManagerDataSource">
<property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
<property name="url" value="jdbc:mysql://localhost:3306/board"></property>
<property name="username" value="계정"></property>
<property name="password" value="계정비번"></property>
</bean>

<bean id="factory" class="org.mybatis.spring.SqlSessionFactoryBean">
<property name="dataSource" ref="dataSource"></property>
<property name="configLocation" value="classpath:/mybatis-config.xml"</property>
<property name="mapperLocations" value="classpath:/mappers/**/*Mapper.xml"></property>
</bean>

<bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate" destroy-method="clearCache">
<constructor-arg name="sqlSessionFactory" ref="factory"></constructor-arg>
</bean>

<context:component-scan base-package="com.board.killi8n.persistance"></context:component-scan>
<context:component-scan base-package="com.board.killi8n.service"></context:component-scan>
```

저희는 데이터베이스명으로 board를 사용하겟습니다. 다른걸로 해주셔도 무방합니다.

factory빈에 써놓은대로, mybatis config파일과 mapper파일을 일단 빈파일이라도 만들어놓으러 갑시다.

# src/main/resources/mybatis-config.xml

위 경로대로, mybatis-config.xml을 생성해줍시다!

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>

</configuration>
```

일단 컨피그를 빈 파일로 생성해놓은뒤,

# src/main/resources/mappers/BoardMapper.xml

위 경로대로 mapper파일을 생성해주세요

```xml
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="xom.board.killi8n.mappers.BoardMapper">

</mapper>
```

그리고 패키지들을 생성해주세요
persistance와 service패키지를 생성해주시면 됩니다.

# servlet-context.xml

그리고 컨트롤러 패키지를 하나 따로 빼주기위해 서블릿컨텍스트를 건들여 볼까요?

```xml
<context:component-scan base-package="com.board.killi8n.controllers" />
```

위와같이 패키지를 하나 만들어주시고, 컨트롤러를 스캔하게 합니다!

그리고 기존의 컨트롤러들은 지워줍시다! 패키지도 모두 지워줍니다.

# BoardController.java

컨트롤러 패키지에 BoardController라는 이름의 클래스를 하나 생성해줍시다.
이 파일이 바로 컨트롤러 역할을 할 것입니다.

```java
package com.board.killi8n.controllers;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

@Controller
public class BoardController {

@RequestMapping(value="/", method=RequestMethod.GET)
public String boardIndex() {
return "home";
}

}
```

일단 위와같이 구성을 시켜주세요!
그리고 서버와 함께 실행 시켜봅시다!

일단 서버를 작동하면, 루트 패스가 아닌 보기싫은 패스가 하나 더 붙어있을것입니다!
루트 패스로 시작하기 위해 STS의 서버 탭으로 가서, Modules탭으로 간뒤, Path를 / 로 바꾸어줍시다!

자 이제 다시 실행시키면 루트패스로 경로를 찾아갈수 있죠?

# Bootstrap

[https://drive.google.com/drive/folders/1rrA9AjZjXECjPNEkFPif7lWVjW6my64-](https://drive.google.com/drive/folders/1rrA9AjZjXECjPNEkFPif7lWVjW6my64-)

일단 위 링크에서 부트스트랩 관련 자료들을 모두 다운받아주세요!

그리고 static폴더에 있는 모든 파일

### src/main/webapp/resources

경로에 모두 넣어줍시다.

include 파일들을

### WEB-INF/views

아래에 include라는 폴더를 만들고 모두 넣어줍시다.

그리고 home.jsp를 지우고, index.jsp를 만들어줍시다!

```html
<%@include file="./include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<%@include file="./include/footer.jsp"%>
```

다음과 같이 index.jsp를 재구성해주세요! bootstrap템플릿의 footer와 header를 포함한 형태입니다.

한글 깨짐 방지를 위해 UTF-8 charset태그도 추가해줍시다

아참, 그리고 이제 home.jsp가 사라졌으니 컨트롤러에서 "home"대신 "index"를 리턴해주어야 겠죠?

자 그럼 이제 저희의 게시판을 디자인해 볼까요?

# include/header.jsp

```html
(...)
<li><a href="/resources/documentation/index.html"><i class="fa fa-book"></i> <span>Documentation</span></a></li>
<li><a href="/board"><i class="fa fa-book"></i> <span>BBS</span></a></li>
(...)
```

header.jsp에서 Documentation이라는 글자를 검색해보시면 다음과 같은 부분이 있을겁니다.

그 밑에 BBS라는 이름으로 게시판 링크를 하나 달아줍시다!

href의 경로를 보시면 /board이므로 컨트롤러에서 경로를 추가 주세요

```java
@RequestMapping(value="/board", method=RequestMethod.GET)
public String boardIndex() {
return "board/boardIndex";
}
```

위와같이 /index로 경로만 바꾸어주세요!

이제 view파일을 만들어야 겟죠?



# views/boardIndex.jsp

위 경로로 jsp파일을 생성해주세요.

```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<style>
.table {
font-weight: 500;
font-size: 1.5rem;
}

</style>

<div class="container">
<div class="col-md-12">
<h1 style="margin-bottom: 2rem;">게시판</h1>
<a href="/board/write" class="btn btn-primary pull-right" style="margin-bottom: 1rem;">글 작성하기</a>
<table class="table table-bordered table-hover" style="width: 100%;">
<thead>
<tr>
<th>게시글 번호</th>
<th>제목</th>
<th>올린 날짜</th>
<th>글쓴이</th>
</tr>
</thead>
<tbody>
<tr style="height: 1.5rem">
<td>1</td>
<td>안녕하세요?</td>
<td>2018-05-13</td>
<td>killi8n</td>
</tr>
</tbody>
</table>
</div>
</div>

<%@include file="../include/footer.jsp"%>
```

일단 디자인이므로, mock 결과물을 넣어봅시다!



그럴싸 한가요?

뭐.. css는 입맛에 맞추어서 다양하게 변경해주세요!



자 이제 글쓰는 페이지도 한번 디자인해봅시다!

# BoardController.java

```java
@RequestMapping(value="/board/write", method=RequestMethod.GET)
public String boardWrite() {
return "board/boardWrite";
}
```

컨트롤러에 위와 같은 경로를 추가해주세요!



# views/boardWrite.jsp

```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<div class="container">
<form action="/board/write" method="post" style="margin-top: 3rem; margin-bottom: 3rem;">
<div class="form-group">
<label for="exampleInputEmail1">글 제목</label> <input name="title"
type="text" class="form-control" id="exampleInputEmail1"
placeholder="title">
</div>
<div class="form-group">
<label for="exampleInputPassword1">글 작성자</label> <input
type="text" class="form-control" id="exampleInputPassword1" name="writer"
placeholder="author">
</div>
<div class="form-group">
<label for="content">내용</label>
<textarea id="content" class="form-control" rows="3" placeholder="글내용" name="content"></textarea>
</div>

<button type="submit" class="btn btn-default">작성하기</button>
</form>
</div>
<%@include file="../include/footer.jsp"%>
```

자 이제 작성하는 폼과 보여주는 리스트를 작성 했으므로, 일단 상세정보 뷰는 접어두고, 컨트롤러부터 작성하러 갑시다!

# BoardController.java

```java
@RequestMapping(value="/board/write", method=RequestMethod.POST)
public String boardWriteAction() {
return "redirect:/board";
}
```
일단 위와같이 작성해두고, 서비스와 퍼시스턴스 패키지를 작성하러 갑시다!


# service/BoardService.java

```java
package com.board.killi8n.service;

import java.util.List;

import com.board.killi8n.domain.BoardVO;



public interface BoardService {

public List<BoardVO> GetAllBoardList();

public void CreateBoard(BoardVO boardVO);

public BoardVO ReadBoard(int bno);

public void UpdateBoard(BoardVO boardVO);

public void DeleteBoard(int bno);
}


```

인터페이스 파일로 작성해주세요!

자
, 이제 boardvo파일을 만들어야겟죠?

domain 패키지를 생성해주신후, BoardVO라는 클래스를 하나 만들어줍시다!

# domain/BoardVO.java

```java
package com.board.killi8n.domain;

import java.util.Date;

public class BoardVO {
private int bno;
private String title;
private String content;
private String writer;
private Date regDate;
private int viewCount;
public int getBno() {
return bno;
}
public void setBno(int bno) {
this.bno = bno;
}
public String getTitle() {
return title;
}
public void setTitle(String title) {
this.title = title;
}
public String getContent() {
return content;
}
public void setContent(String content) {
this.content = content;
}
public String getWriter() {
return writer;
}
public void setWriter(String writer) {
this.writer = writer;
}
public Date getRegDate() {
return regDate;
}
public void setRegDate(Date regDate) {
this.regDate = regDate;
}
public int getViewCount() {
return viewCount;
}
public void setViewCount(int viewCount) {
this.viewCount = viewCount;
}
}

```

이제 DAO Interface 파일도 만들어볼까요?

# persistance/BoardDAO

```java
package com.board.killi8n.persistance;

import java.util.List;

import com.board.killi8n.domain.BoardVO;



public interface BoardDAO {
public List<BoardVO> GetAllBoardList();
public void CreateBoard(BoardVO boardVO);
public BoardVO ReadBoard(int bno);
public void UpdateBoard(BoardVO boardVO);
public void DeleteBoard(int bno);
}

```

이또한 인터페이스 파일입니다.

이제 이 두 인터페이스를 상속받는 클래스들을 만들어봅시다!

# persistance/BoardDAOImpl

```java
package com.board.killi8n.persistance;

import java.util.List;

import javax.inject.Inject;

import org.apache.ibatis.session.SqlSession;
import org.springframework.stereotype.Repository;

import com.board.killi8n.domain.BoardVO;

@Repository
public class BoardDAOImpl implements BoardDAO {

@Inject
SqlSession session;
private static final String NAME_SPACE = "com.board.killi8n.mappers.BoardMapper";
@Override
public List<BoardVO> GetAllBoardList() {
// TODO Auto-generated method stub
return null;
}

@Override
public void CreateBoard(BoardVO boardVO) {
// TODO Auto-generated method stub

}

@Override
public BoardVO ReadBoard(int bno) {
// TODO Auto-generated method stub
return null;
}

@Override
public void UpdateBoard(BoardVO boardVO) {
// TODO Auto-generated method stub

}

@Override
public void DeleteBoard(int bno) {
// TODO Auto-generated method stub

}

}

```

자 이제 구현해야할 메소드들이 많이 있죠?
구현하기 위해서는 mapper파일과 연동해야 하기 때문에, mapper파일을 작성하러 갑시다

# mappers/BoardMapper.xml

```xml
<!DOCTYPE mapper
PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.board.killi8n.mappers.BoardMapper">
<select id="selectBoardListAll" resultType="com.board.killi8n.domain.BoardVO">
<![CDATA[
SELECT * FROM BOARD WHERE bno > 0 ORDER BY bno DESC;
]]>

</select>

<update id="updateBoard">
UPDATE board SET title = #{title}, content =
#{content} WHERE bno = #{bno};
</update>

<delete id="deleteBoard">
DELETE FROM board WHERE bno = #{bno};
</delete>

<insert id="createBoard">
INSERT INTO board (title, content, writer) VALUES
(#{title}, #{content}, #{writer});
</insert>

<select id="selectBoard" resultType=""com.board.killi8n.domain.BoardVO">
SELECT * FROM board WHERE
bno = #{bno};
</select>
</mapper>
```

CDATA를 쓰는 이유는, > , >= , < , <= 와 같이 비교 문자를 사용할때 태그와 같이 파싱이 될 우려가 있기 때문입니다.

위와같이 crud 하는 작업들을 완료해줍시다.

다시 daoImpl파일로 돌아가죠.

# persistance/BoardDAOImpl

```java
package com.board.killi8n.persistance;

import java.util.List;

import javax.inject.Inject;

import org.apache.ibatis.session.SqlSession;
import org.springframework.stereotype.Repository;

import com.board.killi8n.domain.BoardVO;

@Repository
public class BoardDAOImpl implements BoardDAO {

@Inject
SqlSession session;
private static final String NAME_SPACE = "com.board.killi8n.mappers.BoardMapper";
@Override
public List<BoardVO> GetAllBoardList() {
// TODO Auto-generated method stub
return session.selectList(NAME_SPACE + ".selectBoardListAll");
}

@Override
public void CreateBoard(BoardVO boardVO) {
// TODO Auto-generated method stub
session.insert(NAME_SPACE + ".createBoard", boardVO);
}

@Override
public BoardVO ReadBoard(int bno) {
// TODO Auto-generated method stub
return session.selectOne(NAME_SPACE + ".selectBoard", bno);
}

@Override
public void UpdateBoard(BoardVO boardVO) {
// TODO Auto-generated method stub
session.update(NAME_SPACE + ".updateBoard", boardVO);
}

@Override
public void DeleteBoard(int bno) {
// TODO Auto-generated method stub
session.delete(NAME_SPACE + ".deleteBoard", bno);
}

}

```

메소드를 구현해줍시다.


# service/BoardServiceImpl

```java
package com.board.killi8n.service;

import java.util.List;

import javax.inject.Inject;

import org.springframework.stereotype.Service;

import com.board.killi8n.domain.BoardVO;
import com.board.killi8n.persistance.BoardDAOImpl;

@Service
public class BoardServiceImpl implements BoardService {

@Inject
BoardDAOImpl dao;
@Override
public List<BoardVO> GetAllBoardList() {
// TODO Auto-generated method stub
return dao.GetAllBoardList();
}

@Override
public void CreateBoard(BoardVO boardVO) {
// TODO Auto-generated method stub
dao.CreateBoard(boardVO);
}

@Override
public BoardVO ReadBoard(int bno) {
// TODO Auto-generated method stub
return dao.ReadBoard(bno);
}

@Override
public void UpdateBoard(BoardVO boardVO) {
// TODO Auto-generated method stub
dao.UpdateBoard(boardVO);
}

@Override
public void DeleteBoard(int bno) {
// TODO Auto-generated method stub
dao.DeleteBoard(bno);
}

}

```

서비스 impl파일도 다음과 같이 구현해줍시다!

자, 이제 데이터 액세스 메소드를 모두 작성했으니,
컨트롤러에서 inject 해볼까요?


# controller/BoardController.java

```java
package com.board.killi8n.controllers;

import javax.inject.Inject;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.board.killi8n.service.BoardServiceImpl;

@Controller
public class BoardController {
@Inject
BoardServiceImpl service;
@RequestMapping(value="/", method=RequestMethod.GET)
public String index() {
return "index";
}
@RequestMapping(value="/board", method=RequestMethod.GET)
public String boardIndex() {
return "board/boardIndex";
}
@RequestMapping(value="/board/write", method=RequestMethod.GET)
public String boardWrite() {
return "board/boardWrite";
}
@RequestMapping(value="/board/write", method=RequestMethod.POST)
public String boardWriteAction() {
return "redirect:/board";
}
}

```

이제 writeAction메소드를 구현해봅시다

```java
@RequestMapping(value="/board/write", method=RequestMethod.GET)
public String boardWrite(BoardVO vo, Model model) {
try {
service.CreateBoard(vo);
model.addAttribute("message", "Success!");
} catch(Exception e) {
e.printStackTrace();
}
return "board/writeSuccess";
}
```

성공 했다는 페이지를 만들어 주는게 나을거같아, 수정하였습니다.
뷰에서 writeSuccess.jsp를 만들어줍시다!

# views/board/writeSuccess.jsp


```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>


<div class="container">
<div class="jumbotron">
<h1>${message }</h1>
<a href="/board" class="btn btn-primary">게시판으로 이동하기</a>
</div>
</div>

<%@include file="../include/footer.jsp"%>
```

# 데이터베이스 구조

참,, 이것을 처음으로 다뤘어야 하는데, 일단 데이터베이스 구조는 다음과 같습니다!

```sql
create database board;

create table board.board (
bno int not null auto_increment primary key,
title varchar(200) not null,
content text,
writer varchar(50) not null,
regDate timestamp not null default now(),
viewCount int default 0
);
```

그냥 복사붙여넣기해서 mysql에 테이블을 생성해주세요!

자 이제 글을 써보세요!
mysql 테이블에서 select할수 있죠?

그럼 이제 테이블에 들어간 글들을 보여줘볼까요?


# views/board/boardIndex.jsp

```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<style>
.table {
font-weight: 500;
font-size: 1.5rem;
}
</style>

<div class="container">

<div class="col-md-12">
<h1 style="margin-bottom: 2rem;">게시판</h1>
<a href="/board/write" class="btn btn-primary pull-right"
style="margin-bottom: 1rem;">글 작성하기</a>
<table class="table table-bordered table-hover" style="width: 100%;">
<thead>
<tr>
<th>게시글 번호</th>
<th>제목</th>
<th>올린 날짜</th>
<th>글쓴이</th>
</tr>
</thead>
<tbody>
<c:forEach items="${boards }" var="board">
<tr style="height: 1.5rem">
<td>${board.bno }</td>
<td>${board.title }</td>
<td>${board.regDate }</td>
<td>${board.writer }</td>
</tr>
</c:forEach>


</tbody>
</table>
</div>
</div>

<%@include file="../include/footer.jsp"%>
```

c taglib을 사용하여 foreach문을 사용해서 돌려주었습니다.
이제 글 목록도 잘 나타나죠?

그렇다면 이제 세부 글을 볼수있는 페이지를 만들어봅시다.

# views/board/boardDetail.jsp

```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<div class="container">
<table class="table table-bordered table-hover" style="margin-top: 1rem; margin-bottom: 1rem">
<tr>
<td>글 번호</td>
<td>1</td>
</tr>
<tr>
<td>글 제목</td>
<td>글제목입니다.</td>
</tr>
<tr>
<td>작성 날짜</td>
<td>2013-05-09</td>
</tr>
<tr>
<td>글 내용</td>
<td>내용이 들어갑니다.</td>
</tr>
</table>
</div>
<%@include file="../include/footer.jsp"%>
```

임시로 컨트롤러에 매핑을 해봅시다!

```java
@RequestMapping(value="/board/{id}", method=RequestMethod.GET)
public String boardDetail(@PathVariable int id) {
return "board/boardDetail";
}
```

그다음에 서버로 작동하여 들어가보세요 모양이 괜찮은지 확인해보세요!

이제 가짜글이 아닌 진짜글을 봐야겠죠?

```java
@RequestMapping(value="/board/{id}", method=RequestMethod.GET)
public String boardDetail(@PathVariable int id, BoardVO vo, Model model) {
try {
vo = service.ReadBoard(id);
model.addAttribute("vo", vo);
} catch(Exception e) {
e.printStackTrace();
}
return "board/boardDetail";
}
```

위와같이 컨트롤러를 작성한후 뷰를 수정해봅시다!


```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<div class="container">
<table class="table table-bordered table-hover" style="margin-top: 1rem; margin-bottom: 1rem">
<tr>
<td>글 번호</td>
<td>${vo.bno }</td>
</tr>
<tr>
<td>글 제목</td>
<td>${vo.title }</td>
</tr>
<tr>
<td>글 쓴이</td>
<td>${vo.writer }</td>
</tr>
<tr>
<td>작성 날짜</td>
<td>${vo.regDate }</td>
</tr>
<tr>
<td>글 내용</td>
<td>${vo.content }</td>
</tr>
</table>
</div>
<%@include file="../include/footer.jsp"%>
```

아 그리고 bbs 목록페이지에서 link를 걸어줘야겠죠?

# view/board/boardIndex.jsp

```html
(...)
<td><a href="/board/${board.bno }"> ${board.title } </a></td>
(...)
```

위와같이 a 태그를 달아줍시다!

자 그럼이제 읽기는 모두 완료가되었군요!

이제 수정과 삭제를 해볼까요?

# views/board/boardDetail.jsp

```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<div class="container">
<table class="table table-bordered table-hover" style="margin-top: 1rem; margin-bottom: 1rem">
<tr>
<td>글 번호</td>
<td>${vo.bno }</td>
</tr>
<tr>
<td>글 제목</td>
<td>${vo.title }</td>
</tr>
<tr>
<td>글 쓴이</td>
<td>${vo.writer }</td>
</tr>
<tr>
<td>작성 날짜</td>
<td>${vo.regDate }</td>
</tr>
<tr>
<td>글 내용</td>
<td>${vo.content }</td>
</tr>
</table>
<a href="/board/update/${vo.bno }" class="btn btn-warning pull-right" style="margin-left: 1rem;">수정하기</a>
<a href="/board/remove/${vo.bno }" class="btn btn-danger pull-right">삭제하기</a>
</div>
<%@include file="../include/footer.jsp"%>
```


위
와같이 수정과 삭제버튼을 만들고 수정과 삭제를 수행할 링크로 연결해줍시다!

일단 수정같은 경우는 주소 마지막에 쿼리를 달아 주었습니다.
따라서 id 쿼리가 있는경우는 수정이라고 인식하고, 없는경우는 initial Writing 즉, 처음 글을 작성하는 것이지요.

이제 컨트롤러 파일을 수정해볼까요?

```java
@RequestMapping(value="/board/write", method=RequestMethod.GET)
public String boardWrite(String actionState, Model model) {
actionState = "write";
model.addAttribute("actionState", actionState);
return "board/boardWrite";


}

@RequestMapping(value="/board/write", method=RequestMethod.POST)
public String boardWriteAction(BoardVO vo, Model model) {

try {
service.CreateBoard(vo);
model.addAttribute("message", "Success!");
} catch(Exception e) {
e.printStackTrace();
}

return "board/writeSuccess";
}
```

write부분에서 actionState라는 속성을 모델에 전달해줍니다.
actionState로 update와 구분하기위해 form의 action주소를 지정하였습니다.

# views/board/boardWrite.jsp

```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>

<div class="container">
<c:choose>
<c:when test="${empty vo }">
<form action="/board/${actionState }" method="post"
style="margin-top: 3rem; margin-bottom: 3rem;">
</c:when>
<c:otherwise>
<form action="/board/${actionState }/${vo.bno}" method="post"
style="margin-top: 3rem; margin-bottom: 3rem;">
</c:otherwise>
</c:choose>

<div class="form-group">
<label for="exampleInputEmail1">글 제목</label>
<c:choose>
<c:when test="${empty vo }">
<input name="title" type="text" class="form-control"
id="exampleInputEmail1" placeholder="title">
</c:when>
<c:otherwise>
<input type="hidden" name="bno" value="${vo.bno }"/>
<input name="title" type="text" class="form-control"
id="exampleInputEmail1" placeholder="title" value="${vo.title }">
</c:otherwise>
</c:choose>

</div>
<div class="form-group">
<c:choose>
<c:when test="${empty vo }">
<label for="exampleInputPassword1">글 작성자</label> <input type="text"
class="form-control" id="exampleInputPassword1" name="writer"
placeholder="author">
</c:when>
<c:otherwise>
<label for="exampleInputPassword1">글 작성자</label> <input type="text"
class="form-control" id="exampleInputPassword1" name="writer"
placeholder="author" value="${vo.writer }" disabled>
</c:otherwise>
</c:choose>

</div>
<div class="form-group">

<c:choose>
<c:when test="${empty vo }">
<label for="content">내용</label>
<textarea id="content" class="form-control" rows="3"
placeholder="글내용" name="content"></textarea>
</c:when>
<c:otherwise>
<label for="content">내용</label>
<textarea id="content" class="form-control" rows="3"
placeholder="글내용" name="content">${vo.content }</textarea>
</c:otherwise>
</c:choose>
</div>

<c:choose>
<c:when test="${empty vo }">
<button type="submit" class="btn btn-default">작성하기</button>
</c:when>
<c:otherwise>
<button type="submit" class="btn btn-default">수정하기</button>
</c:otherwise>
</c:choose>

</form>
</div>
<%@include file="../include/footer.jsp"%>
```

이 뷰에서는 c taglib을 사용하여 전달된 vo가 있으면 업데이트이고, 아니면 처음 작성하는 것으로 구분하여 뷰를 작성하였습니다.  
업데이트시에는 hidden 속성으로 bno값을 전달해주었습니다.
그리고 actionState로 form의 action주소를 다르게 해주었죠.

update컨트롤러를 한번 살펴볼까요?

```java
@RequestMapping(value="/board/update/{id}", method=RequestMethod.POST)
public String boardUpdateAction(Model model, BoardVO vo, @PathVariable int id) {
try {
service.UpdateBoard(vo);
model.addAttribute("message", "updated!");
} catch(Exception e) {
e.printStackTrace();
}
return "board/writeSuccess";
}

@RequestMapping(value="/board/update/{id}", method=RequestMethod.GET)
public String boardUpdate(Model model, BoardVO vo, @PathVariable int id, String actionState) {
try {
actionState = "update";
vo = service.ReadBoard(id);
model.addAttribute("vo", vo);
model.addAttribute("actionState", actionState);
} catch(Exception e) {
e.printStackTrace();
}
return "board/boardWrite";
}
```

딱히 뒤Pathvariable이 필요하진 않지만, 일단 만들어두었습니다.
get일때는 read로 vo를 전달하고, post일때는 실질적으로 update해주고 success페이지로 넘겨줍니다.

자 이제 수정이 잘되죠?

마지막 삭제를 한번 해봅시다!

# views/board/boardDetail.jsp

```html
(...)
<a href="/board/remove/${vo.bno }" class="btn btn-danger pull-right">삭제하기</a>
(...)
```

아까 디테일 페이지에서 이렇게 링크를 주었죠?

일단 삭제를 할것인지 확인하는 페이지를 만들어 주겠습니다.

컨트롤러에서 매핑부터 하죠!

```java
@RequestMapping(value="/board/remove/{id}", method=RequestMethod.GET)
public String boardDetail(@PathVariable int id, Model model) {

try {
model.addAttribute("confirmMessage", "정말로 " + id + "번 게시물을 삭제할까요?");

} catch(Exception e) {
e.printStackTrace();

}

return "board/removeConfirm";
}
```

이렇게 메시지를 모델로 전달해줍니다.

뷰를 만들어볼까요?

# views/board/removeConfirm.jsp

```html
<%@include file="../include/header.jsp"%>
<%@ page language="java" contentType="text/html; charset=UTF-8"
pageEncoding="UTF-8"%>
<%@taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>


<div class="container">
<div class="jumbotron">
<h2>${confirmMessage }</h2>
<button type="button" class="btn btn-danger" onclick="deleteAction();">삭제하기</button>
<button onclick="goBack();" type="button" class="btn btn-primary">돌아가기</button>
</div>
<form id="deleteForm" style="display: none;">
<input type="hidden" name="bno" value="${bno }">
</form>
</div>

<script>
function goBack() {
window.history.back();
}

function deleteAction() {
var form = document.getElementById("deleteForm");
form.action = "/board/delete/${bno}";
form.method = "post";
form.submit();

}
</script>
<%@include file="../include/footer.jsp"%>
```

히든폼 방식을 사용하였습니다.
돌아가기를 누르면 자바스크립트 함수를 불러 back\(\)해버리고,
삭제 하기를 누르면 폼을 실행하는 함수를 호출합니다.

자, 마지막으로 이제 컨트롤러에서 delete를 구현해볼까요?

```java
@RequestMapping(value="/board/delete/{id}", method=RequestMethod.POST)
public String boardRemove(@RequestParam(value="bno", defaultValue="0", required=false) int bno, Model model)
{
try {
service.DeleteBoard(bno);
model.addAttribute("message", "removed!");

} catch(Exception e) {
e.printStackTrace();
}

return "board/writeSuccess";
}
```

form에서 넘어온 값을 @RequestParam으로 받아서, mybatis 매퍼에 넘겨줍니다!

딱히 특별한 것은 없습니다.



자.. 이제 기본적인 CRUD가 모두 완성되었네요!

뭐, 비밀번호도 걸려있지않고, 회원제 게시판도 아니기때문에 매우 간단하죠?

많은 도움이 되었길 바랍니다!

> ### 자료는 모두 깃헙 저장소에 있습니다. 링크는 아래와 같습니다.
https://github.com/killi8n/SpringBBSMVC
