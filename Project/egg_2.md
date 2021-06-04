# egg 프로젝트

## maven 프로젝트 구성
- 완전 개인 취향으로 구성할 것이기 때문에 IntelliJ를 이용한다.
- [File] - [New] - [Project] 메뉴를 선택해서 Maven 프로젝트를 생성한다.
- archetype은 선택하지 않아도 기본적으로 잘 구성된다.
- 적절한 이름과 프로젝트의 위치를 지정하고, GroupId, ArtifactId, Version도 적절히 입력한다.
- [Finish] 하면 다음과 같은 구조의 프로젝트가 생성된다.

```
+- egg
 +- src
  +- main
   +- java
   +- resources
  +- test
   +- java
  +- pom.xml
```

- 가장 기본. 완전 밑바닥이다.
- 디렉토리 구조만 잡혀 있고 그나마 볼만한 파일이라곤 pom.xml뿐인데 그마저도 별로 볼게 없다. 앞으로 채워나갈 예정.
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.scott</groupId>
    <artifactId>egg</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>14</maven.compiler.source>
        <maven.compiler.target>14</maven.compiler.target>
    </properties>

</project>
```

## git 관리
- 개별 브랜치로 설정들을 관리하기 위해서 일단 git 저장소로 만든다.
- [VCS] - [Create Git Repository] 메뉴를 선택하고, 현재 프로젝트 디렉토리를 선택해서 현재 프로젝트를 git 저장소로 만든다.
- 보이지는 않겠지만 .git 디렉토리가 생성되어 이제 git 관리가 가능하다.
- 실제 프로젝트에는 여러 디렉토리와 pom.xml 파일이 있지만 현재 관리가 가능한 것은 pom.xml 파일 뿐이다.
- pom.xml 파일을 우클릭하여 [Git] - [Add] 선택하고 git의 관리를 받는 파일로 지정하자.
- 이후 첫번째 커밋을 해보자.
- 커밋 메세지는 Maven Project initializing.

## github 원격 저장소 관리
- 내 로컬에만 가지고 있으면 언제든지 날려먹을 수 있다.
- 원격 저장소를 두고 원격 저장소에도 저장한다.
- 원격 저장소는 github에 두자.
- 프로젝트 네비게이션에서 우클릭하여 [Git] - [Manage Remotes] 를 선택한다.
- 원격 저장소의 URL을 설정하도록 되어 있는데 일단은 원격 저장소가 없는 상태이니 github로 가서 원격 저장소를 만들자.
- github에 접속하고 어느 경로로든 접근해서 create new repository를 하자.
- 적절한 이름을 선택하고 아직은 공개하기 좀 아쉬우니 `private`으로 설정할까?
    - 나중에 repository 설정을 통해 public 으로 변경할 수도 있고 다른 곳으로 옮길 수도 있다.
- 아래 `Add a README file`, `Add .gitignore`, `Choose a license` 체크박스도 일단 무시하자. 전부 직접 관리할테니.
- 원격 저장소를 만들고 이제 원격 저장소 주소를 확인할 수 있다. 
- 다시 IntelliJ로 돌아가서 이 원격 저장소를 등록한다. 일반적으로 원격 저장소는 `origin`이라는 이름을 쓴다.
- 이제 원격 저장소로 push 할 수 있게 되었다.
- 프로젝트 네비게이션에서 우클릭 하여 [Git] - [Push] 를 선택하고, 확인, 다시 한번 [Push]를 선택한다.
- 다시 github에 생성한 Repository에 가보면 방금 push 한 파일이 올라와있다.
