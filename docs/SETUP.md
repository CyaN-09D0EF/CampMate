# 개발 환경 및 실행 가이드

[프로젝트 소개로 돌아가기](../README.md)

## 준비 사항

- JDK 21
- Apache Tomcat 10.1: `jakarta.servlet`을 사용하므로 Tomcat 9와 호환되지 않습니다.
- MySQL 호환 DB: 클라우드 환경에서는 MariaDB 11.8로 연동을 확인했습니다.
- Eclipse IDE for Enterprise Java and Web Developers: 저장소의 기존 프로젝트 설정을 사용하는 방법입니다.

외부 라이브러리는 `camp/src/main/webapp/WEB-INF/lib/`에 포함되어 있습니다. Bootstrap과 웹 폰트는 CDN을 사용하므로 브라우저의 외부 네트워크 연결이 필요합니다.

## 1. 프로젝트 가져오기

1. 저장소를 복제합니다.

   ```bash
   git clone https://github.com/CyaN-09D0EF/CampMate.git
   cd CampMate
   ```

2. Eclipse의 **File → Import → General → Existing Projects into Workspace**에서 `camp/`를 선택합니다.
3. 프로젝트 JRE를 Java 21로, Targeted Runtime을 Tomcat 10.1로 지정합니다.
4. 프로젝트의 Java Build Path에서 포함된 JAR와 서버 런타임을 확인합니다.

## 2. 데이터베이스 준비

로컬 DB 서버를 시작하고 DB 관리 계정으로 접속합니다.

```sql
CREATE DATABASE camp_DB CHARACTER SET utf8mb4;
```

저장소 루트에서 다음 명령으로 초기 데이터를 가져옵니다. 비밀번호는 명령줄에 쓰지 않고 프롬프트에서 입력합니다.

```bash
mysql -u root -p camp_DB < camp/src/main/webapp/sql/init.sql
```

MariaDB 클라이언트는 `mysql` 대신 `mariadb`를 사용할 수 있습니다. 초기 SQL은 재실행용 마이그레이션이 아니므로 **빈 DB에서 한 번 실행**하세요. 기존 데이터를 삭제해 오류를 해결하지 마세요.

현재 `DBUtil.java`는 `localhost:3306`, DB 이름 `camp_DB`와 접속 계정을 소스에 직접 지정합니다. 로컬 설정과 일치해야 연결됩니다. 환경 변수로 바꾸는 기능은 아직 없으며, 실제 계정 정보는 공개 커밋에 추가하지 마세요.

## 3. 메일 설정 확인

`EmailUtil.java`는 네이버 SMTP의 465번 SSL 포트를 사용하며 계정 정보를 소스에 직접 지정합니다. 현재는 환경 변수로 계정을 주입할 수 없습니다. 실제 이메일 인증 시연에는 유효한 SMTP 계정과 해당 포트의 통신이 필요합니다.

저장소에 포함된 인증 정보의 유효성을 전제로 사용하지 마세요. 실제 계정 정보라면 소유자가 폐기·재발급하고, 별도 개발 작업으로 설정을 외부화해야 합니다. 새 비밀번호를 문서나 Git에 기록하지 마세요.

## 4. 서버 실행

1. Eclipse의 Servers에서 Tomcat 10.1 서버를 만듭니다.
2. **Add and Remove**에서 `camp` 프로젝트를 추가합니다.
3. 서버를 시작하고 `http://localhost:8080/camp/index.jsp`에 접속합니다.

컨텍스트 경로는 프로젝트 설정상 `/camp`입니다. 서버 포트를 바꿨다면 주소에도 반영하세요.

## 5. 동작 확인

| 확인 대상 | 기대 결과 |
| --- | --- |
| `/camp/index.jsp` | 메인 화면과 초기 상품 카드 4개 |
| `/camp/product` | 상품명·가격·이미지가 들어 있는 JSON 배열 |
| `/camp/user/checkId?username=test` | `taken` |
| `/camp/user/checkId?username=unused_demo_id` | 해당 아이디가 DB에 없으면 `available` |
| `/camp/login.jsp`, `/camp/register.jsp` | 각 입력 화면 렌더링 |
| 가입한 계정으로 로그인 | `main.jsp`로 이동하고 사용자 이름 표시 |

상품 API는 DB 오류 시에도 빈 배열을 반환할 수 있습니다. HTTP 200만 확인하지 말고 데이터와 Tomcat 로그를 함께 확인하세요. 초기 `test` 계정은 비밀번호 형식이 로그인 코드와 달라 정상 로그인 시연에 쓸 수 없습니다.

## 준비된 Codex 클라우드 환경

이 저장소의 온보딩으로 준비된 환경에 `/workspace/.campmate/start.sh`가 존재하면 다음 명령으로 로컬 DB와 Tomcat을 시작하고 Java 소스를 컴파일·배포할 수 있습니다.

```bash
bash /workspace/.campmate/start.sh
```

이 도우미는 환경에 설치된 파일이며 저장소에는 포함되지 않습니다. 일반 로컬 환경에서는 앞의 Eclipse 실행 절차를 사용하세요. 도우미의 상품 준비 상태 검사는 초기 상품 4개를 기준으로 합니다.

## 문제 해결

| 증상 | 확인할 내용 |
| --- | --- |
| Servlet 관련 컴파일 오류 | Tomcat 10.1 런타임과 Jakarta API 연결 |
| 상품 카드가 비어 있음 | DB 실행 상태, 접속 설정, `product` 데이터, 서버 로그 |
| DB 접속 거부 | 로컬 계정 인증 방식과 소스의 접속 설정 일치 여부 |
| 메일 발송 실패 | SMTP 인증·SSL·네트워크 연결 및 서버 로그 |
| 검색·일부 메뉴에서 404 | 연결 대상 기능이 아직 구현되지 않은 항목인지 확인 |
