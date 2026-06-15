# 기능 구현 및 단계별 구현 내용

## 3.1 기능 구현 현황

| 요구사항 번호 | 간단 설명 | 관련 소스 | 구현 여부 |
| :--- | :--- | :--- | :--- |
| REQ-001 | [F-01] 텍스트 분석 기능 구현 | src/main/java/com/phishing/engine/MainAnalyzer.java | ☑ 완성 |
| REQ-002 | [F-02] 위험도 분류 기능 구현 | src/main/java/com/phishing/engine/KeywordMatcher.java | ☑ 완성 |
| REQ-003 | [F-03] 결과 출력 기능 구현 (UI 깨짐 버그 수정 완료) | src/main/java/com/phishing/ui/ResultViewTextArea.java | ☑ 완성 |
| REQ-004 | [F-04] 보호자 알림 기능 구현 (로그 강제 적재 버그 수정 완료) | src/main/java/com/phishing/dashboard/GuardianManager.java | ☑ 완성 |
| REQ-005 | [F-05] 시리얼 통신 기능 구현 (NPE 시스템 크래시 버그 수정 완료) | src/main/java/com/phishing/hardware/SerialHandler.java | ☑ 완성 |
| REQ-006 | [F-06] LED 제어 기능 구현 | src/main/java/com/phishing/hardware/DeviceController.java | ☑ 완성 |
| REQ-007 | [F-07] 부저 제어 기능 구현 | src/main/java/com/phishing/hardware/DeviceController.java | ☑ 완성 |
| REQ-008 | [F-08] 키워드 관리 기능 구현 (리스트 실시간 미갱신 버그 수정 완료) | src/main/java/com/phishing/dashboard/KeywordTableContainer.java | ☑ 완성 |
| REQ-009 | [F-09] 사용자 관리 기능 구현 | src/main/java/com/phishing/dashboard/UserManager.java | ☑ 완성 |
| REQ-010 | [F-10] 알림 설정 기능 구현 (임계값 실시간 미반영 버그 수정 완료) | src/main/java/com/phishing/engine/PhishingAnalyzer.java | ☑ 완성 |
| REQ-011 | [F-11] 전송/알림 이력 로깅 기능 구현 | src/main/java/com/phishing/database/LogRepository.java | ☑ 완성 |
| REQ-012 | [F-12] 시리얼 통신 안정성 구현 (포트 중복 개방 충돌 버그 수정 완료) | src/main/java/com/phishing/hardware/SerialSingletonDriver.java | ☑ 완성 |
| REQ-013 | [F-13] 접근성 지원 기능 구현 | src/main/java/com/phishing/ui/AccessibilitySupport.java | ☑ 완성 |
| REQ-014 | [F-14] 데이터 내보내기/백업 기능 구현 | src/main/java/com/phishing/database/DataBackupService.java | ☑ 완성 |
| REQ-015 | [F-15] 관리자 대시보드 구현 (통계 중복 누적 및 한글 깨짐 수정 완료) | src/main/java/com/phishing/dashboard/StatisticsPanel.java | ☑ 완성 |

* **구현 여부 기호 안내:**
    * ☑ 완성: 요구사항을 완전히 충족하는 기능이 구현되어 정상 동작함
    * ◇ 부분완성: 일부 기능은 동작하나 요구사항을 완전히 충족하지 못함
    * ✘ 미완성: 구현되지 않았거나 전혀 동작하지 않음


## 3.2 구현 내용 설명 

본 시스템은 단일 Java Swing 애플리케이션(MainApp.java 중심 구조) 기반으로 구현되었으며,
기능별 모듈은 내부 메서드 및 SQLite + Arduino 연동 로직으로 구성되어 있습니다.

---

## REQ-001: [F-01] 텍스트 분석 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (handleAnalysis 함수)

설명:  
사용자가 입력한 문자열에서 SQLite custom_keywords 테이블을 조회하여 키워드 목록과 가중치를 로딩한 후,  
문자열 포함 여부를 기반으로 점수를 누적 계산하는 방식으로 구현되었습니다.

- DB → dynamicKeywords(HashMap) 로딩
- contains() 기반 키워드 탐지
- 점수 누적 방식으로 위험도 산출

---

## REQ-002: [F-02] 위험도 분류 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (handleAnalysis 함수)

설명:  
누적된 점수를 기준으로 alertThreshold 값과 비교하여 3단계로 분류합니다.

- SAFE: 0 ~ 1
- WARN: 2 ~ threshold 미만
- DANGER: threshold 이상 또는 특정 패턴 탐지 시 강제 상승

---

## REQ-003: [F-03] 결과 출력 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (createResultPanel 함수)

설명:  
CardLayout을 기반으로 MainScreen과 ResultScreen을 전환하여 결과를 출력합니다.  
위험도에 따라 JLabel 색상 및 JTextArea 내용을 동적으로 변경합니다.

---

## REQ-004: [F-04] 보호자 알림 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (saveGuardianAlertLog 함수)

설명:  
사용자 등록 시 user_profiles 테이블에 저장되며,  
위험 탐지 시 guardian_alerts 테이블에 알림 로그를 기록합니다.

---

## REQ-005: [F-05] 시리얼 통신 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (safeSerialSend 함수), serial.ArduinoSerialManager

설명:  
ArduinoSerialManager를 통해 위험 상태 코드(0,1,2)를 시리얼 포트로 전송합니다.  
연결 실패 시 예외 처리를 통해 시스템 중단을 방지합니다.

---

## REQ-006: [F-06] 아두이노 경고 출력 기능
구현 여부: ✅ 완성

관련 소스: Arduino 펌웨어 + safeSerialSend 호출

설명:  
전송된 위험도 코드에 따라 아두이노에서 LED 및 부저를 제어합니다.

- 0: SAFE (초록 LED)
- 1: WARN (노랑 LED)
- 2: DANGER (빨강 + 부저)

---

## REQ-007: [F-07] 키워드 관리 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (openAdminDashboard → 키워드 탭)

설명:  
SQLite custom_keywords 테이블을 기반으로 JTable과 연동하여  
키워드 추가 / 수정 / 삭제 기능을 제공합니다.

---

## REQ-008: [F-08] 사용자 관리 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (user_profiles 관련 처리)

설명:  
사용자 이름과 보호자 정보를 SQLite에 저장하고  
관리자 대시보드에서 조회 및 수정할 수 있도록 구현되었습니다.

---

## REQ-009: [F-09] 알림 임계값 설정 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (JSlider thresholdSlider)

설명:  
JSlider를 통해 alertThreshold 값을 실시간 변경하며  
분석 로직에 즉시 반영됩니다.

---

## REQ-010: [F-10] 로그 저장 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (saveLogToDatabase 함수)

설명:  
분석 결과를 phishing_logs 테이블에 저장하며  
내용, 점수, 위험도, 키워드를 함께 기록합니다.

---

## REQ-011: [F-11] 로그 조회 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (refreshLogTable 함수)

설명:  
SQLite phishing_logs 데이터를 JTable로 출력하여  
관리자 대시보드에서 전체 로그를 확인할 수 있도록 구현되었습니다.

---

## REQ-012: [F-12] 데이터 백업 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (runBackup 함수)

설명:  
SQLite 로그 데이터를 CSV 또는 JSON 파일로 저장하는 기능입니다.

- CSV: 엑셀 분석용
- JSON: 데이터 구조 보존용

---

## REQ-013: [F-13] UI 접근성 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (UI Font 설정 영역)

설명:  
전체 UI에 맑은 고딕 폰트를 적용하고  
컴포넌트 크기 및 색상 대비를 조정하여 가독성을 향상시켰습니다.

---

## REQ-014: [F-14] 관리자 대시보드 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/analysis/MainApp.java (openAdminDashboard 함수)

설명:  
JDialog 기반 관리 화면으로 구성되어 있으며,  
JTabbedPane을 통해 다음 기능을 통합 제공합니다:

- 통계 조회
- 키워드 관리
- 사용자 관리
- 로그 조회
- 임계값 설정

---

## REQ-015: [F-15] Arduino 연동 기능
구현 여부: ✅ 완성

관련 소스: src/main/java/serial/ArduinoSerialManager.java

설명:  
jSerialComm 기반으로 Arduino와 연결하여  
위험 상태 값을 실시간 전송하는 기능입니다.