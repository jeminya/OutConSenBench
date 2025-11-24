# OutConSenBench - 프로젝트 개발 계획

## 프로젝트 개요
Output Context Sensitive XSS 취약점 벤치마크 프로젝트로, 제어 흐름의 다양성, 출력 컨텍스트 중첩의 다양성, sanitizer 적용의 다양성을 모두 고려한 체계적인 테스트 케이스 세트를 제공합니다.

**참고 프로젝트:**
- ConSenBench: ../ConSenBench
- Securibench Micro: [Stanford Securibench](https://github.com/too4words/securibench-micro)
- OWASP Benchmark: [OWASP Benchmark Project](https://github.com/OWASP-Benchmark/BenchmarkJava)

---

## 1. 프로젝트 기본 구조 설정

### 1.1 디렉토리 구조
```
src/main/
├── java/
│   └── com/outconsen/bench/
│       ├── basic/              # 기본 단일 컨텍스트 테스트
│       ├── nested/             # 중첩 컨텍스트 테스트
│       ├── controlflow/        # 제어 흐름 다양성 테스트
│       ├── sanitizer/          # Sanitizer 적용 패턴 테스트
│       ├── combination/        # 복합 시나리오 테스트
│       └── utils/              # 공통 유틸리티
└── webapp/
    ├── WEB-INF/
    │   └── web.xml
    └── index.html              # 테스트 케이스 목록 페이지
```

### 1.2 빌드 설정
- [x] Gradle 프로젝트 기본 구조
- [ ] 의존성 추가:
  - Apache Commons Text (StringEscapeUtils)
  - OWASP Java Encoder
  - OWASP ESAPI
  - Servlet API 4.0.1
- [ ] WAR 빌드 태스크 설정

---

## 2. 출력 컨텍스트 분류 체계

### 2.1 기본 단일 컨텍스트 (basic/)
각 컨텍스트별로 safe/unsafe 버전 작성

#### HTML 컨텍스트
- `BasicHTMLPcdata.java` - PCDATA 영역
- `BasicHTMLComment.java` - HTML 주석
- `BasicHTMLTagName.java` - 태그 이름

#### JavaScript 컨텍스트
- `BasicJSString.java` - JS 문자열 리터럴
- `BasicJSCode.java` - JS 코드 영역
- `BasicJSComment.java` - JS 주석

#### CSS 컨텍스트
- `BasicCSSProperty.java` - CSS 속성 값
- `BasicCSSString.java` - CSS 문자열
- `BasicCSSComment.java` - CSS 주석

#### HTML 속성 컨텍스트
- `BasicAttrUnquoted.java` - 인용부호 없는 속성
- `BasicAttrSingleQuoted.java` - 작은따옴표 속성
- `BasicAttrDoubleQuoted.java` - 큰따옴표 속성

#### URI 컨텍스트
- `BasicURIScheme.java` - URI scheme
- `BasicURIPath.java` - URI path
- `BasicURIQuery.java` - URI query parameter

**예상 테스트 수:** 15개 × 2(safe/unsafe) = 30개

---

### 2.2 중첩 컨텍스트 (nested/)
ConSenBench 스타일의 중첩 패턴

#### 이중 중첩 (Level 2)
- `HTML_TO_JS__ScriptTag.java` - `<script>` 태그 내부
- `HTML_TO_JS__OnEvent.java` - 이벤트 핸들러 (onclick, onerror 등)
- `HTML_TO_CSS__StyleTag.java` - `<style>` 태그 내부
- `HTML_TO_CSS__StyleAttr.java` - style 속성
- `HTML_TO_URI__HrefAttr.java` - href 속성
- `HTML_TO_URI__SrcAttr.java` - src 속성
- `CSS_TO_URI__URLFunction.java` - CSS url() 함수
- `CSS_TO_URI__ImportRule.java` - @import 규칙
- `URI_TO_JS__JavascriptScheme.java` - javascript: scheme
- `URI_TO_HTML__DataScheme.java` - data:text/html scheme
- `JS_TO_HTML__DocumentWrite.java` - document.write()
- `JS_TO_HTML__InnerHTML.java` - element.innerHTML

**예상 테스트 수:** 12개 × 2(safe/unsafe) = 24개

#### 삼중 중첩 (Level 3)
- `HTML_CSS_URI__StyleUrlAttr.java` - `style="background: url(...)"`
- `HTML_JS_URI__OnEventHref.java` - `onclick="location.href='...'"`
- `HTML_JS_HTML__ScriptDocWrite.java` - `<script>document.write('<div>...')</script>`
- `HTML_ATTR_JS__DataAttrEvent.java` - `<div data-value="..." onclick="use(data)">`
- `URI_HTML_JS__DataHtmlScript.java` - `data:text/html,<script>...</script>`
- `CSS_URI_JS__ImportJsScheme.java` - `@import url(javascript:...)`

**예상 테스트 수:** 6개 × 2(safe/unsafe) = 12개

#### 사중 중첩 (Level 4)
- `HTML_ATTR_CSS_URI__StyleInAttr.java` - 속성 내 style 내 url
- `HTML_JS_HTML_JS__NestedDocWrite.java` - 중첩된 document.write
- `URI_HTML_ATTR_JS__ComplexData.java` - data URI 내 복잡한 중첩

**예상 테스트 수:** 3개 × 2(safe/unsafe) = 6개

---

## 3. 제어 흐름 다양성 (controlflow/)

### 3.1 데이터 흐름 복잡도
OWASP Benchmark와 Securibench Micro 스타일

#### 직접 출력
- `DirectOutput.java` - 입력을 바로 출력
```java
String input = request.getParameter("data");
out.println("<div>" + input + "</div>");
```

#### 단일 변수 전달
- `SingleVariablePass.java` - 변수 한 번 거침
```java
String input = request.getParameter("data");
String temp = input;
out.println("<div>" + temp + "</div>");
```

#### 다중 변수 전달
- `MultipleVariablePass.java` - 여러 변수 거침
```java
String input = request.getParameter("data");
String temp1 = input;
String temp2 = temp1;
String temp3 = temp2;
out.println("<div>" + temp3 + "</div>");
```

#### 메소드 호출 (Interprocedural)
- `SingleMethodCall.java` - 단일 메소드 호출
- `ChainedMethodCalls.java` - 메소드 체인 (3단계)
- `RecursiveMethodCall.java` - 재귀 호출

#### 컬렉션을 통한 전달
- `ArrayPass.java` - 배열 사용
- `ListPass.java` - ArrayList 사용
- `MapPass.java` - HashMap 사용
- `SetPass.java` - HashSet 사용

#### 객체 필드를 통한 전달
- `ObjectFieldPass.java` - 객체 필드 저장/조회
- `StaticFieldPass.java` - static 필드 사용

**예상 테스트 수:** 13개 × 2(safe/unsafe) = 26개

---

### 3.2 제어 구조 다양성

#### 조건문
- `IfElseFlow.java` - if-else 분기
- `SwitchCaseFlow.java` - switch-case 분기
- `TernaryOperatorFlow.java` - 삼항 연산자

#### 반복문
- `ForLoopFlow.java` - for 루프
- `WhileLoopFlow.java` - while 루프
- `ForEachFlow.java` - for-each 루프 (컬렉션 순회)

#### 예외 처리
- `TryCatchFlow.java` - try-catch에서 출력
- `FinallyBlockFlow.java` - finally 블록에서 출력
- `ThrowCatchFlow.java` - 예외 던지고 받아서 출력

#### 복합 제어 구조
- `NestedLoopsFlow.java` - 중첩 루프
- `LoopWithConditionFlow.java` - 루프와 조건문 조합
- `MultipleExceptionsFlow.java` - 다중 catch 블록

**예상 테스트 수:** 12개 × 2(safe/unsafe) = 24개

---

### 3.3 고급 데이터 흐름

#### Predicate/Validation
- `PredicateBasedFlow.java` - Predicate로 필터링 후 출력
- `ValidationBeforeOutput.java` - validation 로직 후 출력
- `ConditionalSanitization.java` - 조건부 sanitization

#### String 연산
- `StringConcatenation.java` - 문자열 연결
- `StringBuilderFlow.java` - StringBuilder 사용
- `StringFormatFlow.java` - String.format() 사용
- `StringReplaceFlow.java` - replace/replaceAll 사용

#### Type Casting/Conversion
- `ObjectToStringCast.java` - Object → String 캐스팅
- `PrimitiveToString.java` - primitive → String 변환

#### Reflection
- `ReflectionFieldAccess.java` - Reflection으로 필드 접근
- `ReflectionMethodCall.java` - Reflection으로 메소드 호출

**예상 테스트 수:** 11개 × 2(safe/unsafe) = 22개

---

## 4. Sanitizer 적용 패턴 (sanitizer/)

### 4.1 Sanitizer 종류별 테스트

#### Apache Commons Text
- `CommonsTextHtmlEscape.java` - StringEscapeUtils.escapeHtml4()
- `CommonsTextJsEscape.java` - StringEscapeUtils.escapeEcmaScript()
- `CommonsTextCssEscape.java` - StringEscapeUtils.escapeCss()
- `CommonsTextXmlEscape.java` - StringEscapeUtils.escapeXml()

#### OWASP Java Encoder
- `EncoderForHtml.java` - Encode.forHtml()
- `EncoderForJavaScript.java` - Encode.forJavaScript()
- `EncoderForCss.java` - Encode.forCssString()
- `EncoderForUri.java` - Encode.forUriComponent()

#### OWASP ESAPI
- `ESAPIHtmlEncoder.java` - ESAPI.encoder().encodeForHTML()
- `ESAPIJsEncoder.java` - ESAPI.encoder().encodeForJavaScript()
- `ESAPICssEncoder.java` - ESAPI.encoder().encodeForCSS()

#### Custom Sanitizers
- `CustomBlacklist.java` - 블랙리스트 기반 필터링
- `CustomWhitelist.java` - 화이트리스트 기반 필터링
- `CustomRegexFilter.java` - 정규식 기반 필터링

**예상 테스트 수:** 14개 (각각 올바른 적용 예제)

---

### 4.2 Sanitizer 오용/부족 패턴

#### 컨텍스트 불일치
- `WrongSanitizerForContext.java` - HTML 컨텍스트에 JS sanitizer 사용
- `PartialSanitization.java` - 중첩 중 일부만 sanitize
- `OrderMismatch.java` - sanitizer 적용 순서 잘못됨

#### Sanitizer 우회
- `DoubleSanitization.java` - 이중 인코딩 문제
- `SanitizeBeforeConcat.java` - sanitize 후 추가 문자열 연결
- `IncompleteBlacklist.java` - 불완전한 블랙리스트

#### Null/Empty 처리
- `NullInputNoSanitizer.java` - null 입력 시 sanitizer 미적용
- `EmptyStringBypass.java` - 빈 문자열 처리 누락

**예상 테스트 수:** 8개

---

### 4.3 올바른 Sanitizer 적용

각 중첩 레벨별로 올바른 sanitizer chain 적용 예제

#### 이중 중첩
- `CorrectHTML_JS.java` - HTML → JS 올바른 sanitization
- `CorrectHTML_CSS.java` - HTML → CSS 올바른 sanitization
- `CorrectHTML_URI.java` - HTML → URI 올바른 sanitization

#### 삼중 중첩
- `CorrectHTML_CSS_URI.java` - HTML → CSS → URI
- `CorrectHTML_JS_URI.java` - HTML → JS → URI

#### 사중 중첩
- `CorrectHTML_ATTR_CSS_URI.java` - 완전한 체인

**예상 테스트 수:** 6개

---

## 5. 복합 시나리오 (combination/)

실제 애플리케이션 시나리오를 반영한 복합 테스트

### 5.1 실제 패턴 모방
- `UserProfileDisplay.java` - 사용자 프로필 표시
- `CommentSystem.java` - 댓글 시스템
- `SearchResultDisplay.java` - 검색 결과 표시
- `DynamicHTMLGeneration.java` - 동적 HTML 생성
- `JSONResponse.java` - JSON 응답 생성

### 5.2 복잡한 시나리오
- `ComplexControlFlowNested.java` - 제어 흐름 + 중첩 컨텍스트
- `CollectionWithNesting.java` - 컬렉션 순회 + 중첩 출력
- `ConditionalSanitizerNested.java` - 조건부 sanitizer + 중첩
- `MultipleInputsNested.java` - 다중 입력 + 중첩 컨텍스트
- `AsyncDataFlowNested.java` - 비동기 처리 + 중첩

**예상 테스트 수:** 10개 × 2(safe/unsafe) = 20개

---

## 6. 테스트 케이스 명명 규칙

### 형식
```
[Category][Number]_[Description]_[Status].java
```

### 예시
- `Basic01_HTMLPcdata_unsafe.java`
- `Basic01_HTMLPcdata_safe.java`
- `Nested12_HTML_JS_URI_unsafe.java`
- `Nested12_HTML_JS_URI_safe.java`
- `Control05_ChainedMethods_unsafe.java`
- `Control05_ChainedMethods_safe.java`

---

## 7. 공통 유틸리티 (utils/)

### 7.1 기본 클래스
- `TestCaseBase.java` - 모든 테스트의 부모 클래스
- `SanitizerUtils.java` - 다양한 sanitizer 메소드 모음
- `ContextHelper.java` - 컨텍스트 판별 헬퍼

### 7.2 테스트 메타데이터
- `TestMetadata.java` - 각 테스트의 메타정보 (컨텍스트, 예상 결과 등)
- `ExpectedResults.java` - 예상 결과 정의

---

## 8. 웹 인터페이스

### 8.1 인덱스 페이지
- `index.html` - 전체 테스트 케이스 목록
  - 카테고리별 분류
  - 필터링 기능 (safe/unsafe, 컨텍스트별)
  - 각 테스트로 직접 이동 링크

### 8.2 테스트 결과 페이지
- `results.jsp` - 테스트 실행 결과 표시
- `testinfo.jsp` - 개별 테스트 상세 정보

---

## 9. 문서화

### 9.1 프로젝트 문서
- `README.md` - 프로젝트 소개, 빌드 방법, 사용법
- `ARCHITECTURE.md` - 아키텍처 설명
- `TEST_CATEGORIES.md` - 테스트 카테고리 상세 설명

### 9.2 참고 문서
- `REFERENCES.md` - ConSenBench, Securibench, OWASP Benchmark 참고 내용
- `CONTEXT_THEORY.md` - 출력 컨텍스트 이론 정리

---

## 10. 개발 일정 및 우선순위

### Phase 1: 기본 인프라 (1주)
- [x] Gradle 프로젝트 구조
- [ ] 의존성 설정
- [ ] 공통 유틸리티 클래스
- [ ] 웹 애플리케이션 기본 구조
- [ ] 인덱스 페이지

### Phase 2: 기본 단일 컨텍스트 (1주)
- [ ] HTML 컨텍스트 테스트 (6개)
- [ ] JavaScript 컨텍스트 테스트 (6개)
- [ ] CSS 컨텍스트 테스트 (6개)
- [ ] 속성 컨텍스트 테스트 (6개)
- [ ] URI 컨텍스트 테스트 (6개)

### Phase 3: 중첩 컨텍스트 (2주)
- [ ] 이중 중첩 테스트 (24개)
- [ ] 삼중 중첩 테스트 (12개)
- [ ] 사중 중첩 테스트 (6개)

### Phase 4: 제어 흐름 다양성 (2주)
- [ ] 데이터 흐름 테스트 (26개)
- [ ] 제어 구조 테스트 (24개)
- [ ] 고급 데이터 흐름 테스트 (22개)

### Phase 5: Sanitizer 패턴 (1주)
- [ ] Sanitizer 종류별 테스트 (14개)
- [ ] 오용/부족 패턴 (8개)
- [ ] 올바른 적용 (6개)

### Phase 6: 복합 시나리오 (1주)
- [ ] 실제 패턴 모방 (10개)
- [ ] 복잡한 시나리오 (10개)

### Phase 7: 문서화 및 테스트 (1주)
- [ ] README 작성
- [ ] 각 카테고리 문서 작성
- [ ] Gemini 프로젝트와 통합 테스트
- [ ] Tomcat 배포 테스트

---

## 11. 예상 총 테스트 케이스 수

| 카테고리 | Safe | Unsafe | 합계 |
|---------|------|--------|------|
| 기본 단일 컨텍스트 | 15 | 15 | 30 |
| 이중 중첩 | 12 | 12 | 24 |
| 삼중 중첩 | 6 | 6 | 12 |
| 사중 중첩 | 3 | 3 | 6 |
| 제어 흐름 - 데이터 흐름 | 13 | 13 | 26 |
| 제어 흐름 - 제어 구조 | 12 | 12 | 24 |
| 제어 흐름 - 고급 | 11 | 11 | 22 |
| Sanitizer 종류별 | 14 | 0 | 14 |
| Sanitizer 오용 | 0 | 8 | 8 |
| Sanitizer 올바른 적용 | 6 | 0 | 6 |
| 복합 시나리오 | 10 | 10 | 20 |
| **총계** | **102** | **90** | **192** |

---

## 12. 품질 기준

### 각 테스트 케이스는 다음을 만족해야 함:
1. ✅ 독립적으로 실행 가능한 서블릿
2. ✅ 명확한 취약점 존재 여부 (safe/unsafe)
3. ✅ 실행 가능한 코드 (컴파일 및 배포 가능)
4. ✅ 명확한 주석과 설명
5. ✅ 일관된 코드 스타일
6. ✅ 메타데이터 포함 (어떤 컨텍스트, 어떤 제어 흐름 등)

---

## 13. 확장 계획

### 향후 추가 가능 항목:
- SQL Injection 컨텍스트
- LDAP Injection 컨텍스트
- XML/XPath Injection 컨텍스트
- Command Injection 컨텍스트
- JSP 파일 기반 테스트
- 프레임워크별 테스트 (Spring, Struts 등)

---

## References

- [Securibench Micro](https://github.com/too4words/securibench-micro) - Stanford University의 보안 벤치마크
- [OWASP Benchmark](https://github.com/OWASP-Benchmark/BenchmarkJava) - OWASP의 취약점 탐지 도구 벤치마크
- ConSenBench - 출력 컨텍스트 민감 XSS 벤치마크 (../ConSenBench)
