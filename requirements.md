1. 요구사항 분석 및 설계 


## 1. 기능적 요구사항

텍스트 분석 엔진: 사용자가 입력한 문자열에서 위험 키워드(예: 검찰, 이체, 당첨)를 추출하고 위험 점수를 산출해야 함.

실시간 결과 출력: Java GUI 화면에 분석된 위험 지수(안전, 주의, 위험)를 시각적으로 표시해야 함.

원격 알림 시뮬레이션: 위험 상황 감지 시 보호자에게 전송할 알림 메시지 로그를 생성해야 함

+하드웨어 연동 (부가 서비스): 분석 결과 데이터(0, 1, 2 등)를 시리얼 통신을 통해 아두이노로 즉시 전송해야 함.

+물리적 경고 피드백: 전송된 데이터에 따라 아두이노의 LED 색상을 변경(초록, 노랑, 빨강)하고 부저음 발생

| ID | 요구사항 | 우선순위 |
| --- | --- | ---: |
| F-01 | 텍스트 분석 기능: 시스템은 사용자가 입력한 문자열에서 위험 키워드를 탐지하고, 키워드의 가중치 및 빈도를 기반으로 위험 점수를 계산한다. | 높음 |
| F-02 | 위험도 분류 기능: 시스템은 계산된 점수를 기준으로 안전 / 주의 / 위험의 세 단계로 분류한다. | 높음 |
| F-03 | 결과 출력 기능: 시스템은 분석 결과를 Java GUI 화면에 시각적으로 표시하며, 위험 수준에 따라 색상으로 구분한다. | 높음 |
| F-04 | 보호자 알림 기능: 시스템은 위험 상황 감지 시 보호자에게 전달할 알림 메시지 로그를 생성하여 저장한다. | 중간 |
| F-05 | 시리얼 통신 기능: 시스템은 분석 결과를 시리얼 통신(USB)으로 Arduino에 전송한다(데이터: 0=안전,1=주의,2=위험). | 중간 |
| F-06 | LED 제어 기능: Arduino를 통해 위험도에 따라 LED 색상을 변경하여 시각적 경고를 제공한다(0:초록,1:노랑,2:빨강). | 낮음 |
| F-07 | 부저 제어 기능: 위험 상태(2)일 때만 Arduino를 통해 부저음을 발생시켜 청각적 경고를 제공한다. | 낮음 |
| F-08 | 키워드 관리 기능: 관리자는 키워드 추가·수정·삭제 및 가중치 조정이 가능한 UI를 통해 키워드 목록을 관리할 수 있다. | 중간 |
| F-09 | 사용자 관리 기능: 시스템은 사용자 등록·수정·비활성화 기능을 제공하고 사용자별 보호자 정보를 관리한다. | 중간 |
| F-10 | 알림 설정 기능: 사용자는 보호자 알림 수신 여부 및 채널(이메일/전화)과 알림 임계값을 설정할 수 있다. | 중간 |
| F-11 | 전송/알림 이력 로깅: 시스템은 보호자 알림 전송 결과 및 Arduino 전송 이력을 기록하여 조회 가능하게 한다. | 낮음 |
| F-12 | 시리얼 통신 안정성: 시리얼 전송 실패 시 재시도 및 오류 로깅, 타임아웃 처리 로직을 제공한다. | 중간 |
| F-13 | 접근성 지원: 노약자 인식을 위한 큰 글꼴, 높은 색상 대비, 음성 안내 옵션을 제공한다. | 높음 |
| F-14 | 데이터 내보내기/백업: 분석 로그 및 알림 로그를 CSV/JSON 형식으로 내보내기 및 주기적 백업 기능을 제공한다. | 낮음 |
| F-15 | 관리자 대시보드: 전체 시스템 상태(최근 경고, 키워드 통계, 전송 실패 등)를 시각화하여 제공한다. | 낮음 |

## 2. 비기능적 요구사항

반응 속도: 문자 입력 후 분석 및 아두이노 출력까지의 지연 시간이 최소화.

직관적 UI: 노약자 및 일반 사용자가 한눈에 알아볼 수 있도록 큰 글씨와 명확한 색상.

확장성: 새로운 피싱 키워드가 발견될 경우 리스트를 쉽게 업데이트할 수 있는 구조.


3.시스템 설계 

Input: Java GUI를 통한 문자열 입력

Process: Java 분석 로직 (점수 계산 및 등급 분류)

Output 1: Java 화면 결과 메시지 출력

Output 2: Serial 통신을 통해 Arduino로 신호 전달

+Action: Arduino 하드웨어 작동 (LED/부저) 및 보호자 알림 로그 생성


### users 테이블 (사용자 정보)

| 필드명 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY | 사용자 고유 ID |
| name | VARCHAR(200) NOT NULL | 사용자 이름 |
| phone | VARCHAR(50) NULL | 사용자 전화번호(선택) |
| guardian_name | VARCHAR(200) NULL | 보호자 이름 |
| guardian_contact | VARCHAR(255) NULL | 보호자 연락처(전화번호 또는 이메일) |
| created_at | TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP | 등록 시간 |
| updated_at | TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정 시간 |

---

### phishing_keywords 테이블 (위험 키워드 및 가중치)

| 필드명 | 타입 | 설명 |
| --- | --- | --- |
| id | INT UNSIGNED AUTO_INCREMENT PRIMARY KEY | 키워드 고유 ID |
| keyword | VARCHAR(255) NOT NULL UNIQUE | 탐지할 키워드 문자열(예: "검찰", "이체", "당첨") |
| weight | INT NOT NULL DEFAULT 1 | 단어별 가중치 점수(합산용) |
| is_active | TINYINT(1) NOT NULL DEFAULT 1 | 사용 여부 플래그(활성화/비활성화) |
| created_at | TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP | 등록 시간 |
| updated_at | TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP | 수정 시간 |

---

### alert_logs 테이블 (알림 로그)

| 필드명 | 타입 | 설명 |
| --- | --- | --- |
| id | BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY | 로그 고유 ID |
| user_id | BIGINT UNSIGNED NULL | users 테이블 외래키(관련 사용자) |
| input_text | TEXT NOT NULL | 사용자가 입력한 원본 텍스트 |
| total_score | INT NOT NULL | 계산된 총점 |
| risk_level | TINYINT(1) NOT NULL COMMENT '0=안전,1=주의,2=위험' | 최종 등급(0/1/2) |
| detected_keywords | JSON NULL | 감지된 키워드 및 가중치 목록(JSON 형태) |
| notified | TINYINT(1) NOT NULL DEFAULT 0 | 보호자에게 알림 전송 여부(0=미전송,1=전송) |
| created_at | TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP | 로그 생성 시간 |

