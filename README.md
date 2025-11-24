# OutConSenBench 작업 목록

## 프로젝트 개요
- Output Context Sensitive XSS 취약점을 가진 Java EE 서블릿 벤치마크 프로젝트
- Gemini 프로젝트(Tomcat bytecode instruction injection 기반 XSS 검출 도구)에서 테스트할 수 있는 WAR 파일 생성

## 해야 할 작업들

### 1. 프로젝트 구조 설정
- [ ] Gradle 프로젝트 기본 구조 생성
- [ ] src/main/java 디렉토리 생성
- [ ] src/main/webapp 디렉토리 생성
- [ ] src/main/webapp/WEB-INF 디렉토리 생성
- [ ] web.xml 설정 파일 작성

### 2. XSS 취약점 벤치마크 서블릿 작성
- [ ] 다양한 출력 컨텍스트별 XSS 취약점 예제 작성
  - [ ] HTML 컨텍스트 XSS
  - [ ] JavaScript 컨텍스트 XSS
  - [ ] URL 컨텍스트 XSS
  - [ ] CSS 컨텍스트 XSS
  - [ ] HTML 속성 컨텍스트 XSS
- [ ] 각 취약점별로 개별 서블릿 구현

### 3. 테스트 케이스 작성
- [ ] 각 XSS 유형별 테스트 페이지 작성
- [ ] 취약한 코드와 안전한 코드 비교 예제 작성

### 4. 빌드 및 배포
- [ ] WAR 파일 빌드 테스트
- [ ] Gemini 프로젝트와의 통합 테스트
- [ ] Tomcat에서 WAR 파일 배포 테스트

### 5. 문서화
- [ ] README.md 작성 (프로젝트 설명, 빌드 방법, 사용법)
- [ ] 각 취약점 유형에 대한 설명 문서 작성

## 참고사항
- Java 버전: 11
- 서블릿 API: javax.servlet-api 4.0.1
- Gemini 프로젝트 경로: ../Gemini
- ConSenBench 프로젝트 참고: ../ConSenBench

---
## 작업 메모
<!-- 여기에 작업하면서 필요한 메모를 추가하세요 -->
