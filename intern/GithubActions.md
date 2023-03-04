## Github Actions

소프트웨어 workflow를 자동화할 수 있게 도와주는 도구

<br>

### workflow
- test code
- 배포 및 CI/CD
- 스크립트

<br>

### 알아야 할 개념

1. **workflow**

    여러 job으로 구성, event에 의해 trigger될 수 있는 자동화된 프로세스

    파일을 yaml으로 작성
    
    `./github/workflows` 에 저장됨

2. **event**

    workflow를 trigger하는 특정 활동이나 규칙

    특정 브랜치로 push/pull request, 특정 시간대, 외부 이벤트 등..


3. **job**
    
    여러 step으로 구성

    가상 환경의 인스턴스에서 실행됨

    독립적 실행 및 의존 관계 o

4. **step**

    task의 집합

    커맨드 날리거나 action 실행 가능

5. **action**

    workflow의 가장 작은 단위(블럭)

    job 만들기 위해 step을 연결

    재사용 가능한 컴포넌트

6. **runner**

    Github Action Runner 어플리케이션이 설치된 머신으로 workflow가 실행될 인스턴스

    Github-hosted runner / Self-hosted runner

<br>

### Gitbun Actions 생성

1. 코드 작성
2. workflow 정의
    
    `.github/workflows` 폴더 안 `.yml` 파일 생성

    github Repository에서 Actions

    `set up this workflow` 클릭하여 생성


3. 작동 테스트


<br>
[참고문헌]
https://zzsza.github.io/development/2020/06/06/github-action/