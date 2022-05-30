# 리눅스 구조

["실습과 그림으로 배우는 리눅스 구조"](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=181554153) 책을 읽으며 실습을 진행해보자!

<details>
<summary>1장 컴퓨터 시스템의 개요</summary>

### 리눅스의 주요 역할
- OS가 없으면 여러개의 프로세스가 각자 디바이스를 조작하는 코드를 작성해야함
    - 개발 비용이 커짐
    - 모든 개발자가 디바이스 스펙을 알아야 함 
    - ✅ 리눅스에서는 디바이스 드라이버를 통해 각 프로세스가 디바이스에 접근
- 프로세스가 직접 하드웨어에 접근하는 것을 막아야 함
  - CPU에는 커널모드, 사용자 모드 존재
  - ✅ 커널모드 일때만 디바이스에 접근 가능
  - 또한 커널모드에서는?
    - 프로세스 관리, 스케줄링
    - 메모리 관리
  - OS는 커널 + 사용자 모드에서 동작하는 다양한 프로그램
- 프로세스 실행은 다양한 계층 구조를 구성하며 동작
- 저장 장치에 보관된 데이터는 디바이스 드라이버에 직접 요청해 접근 가능하지만, 보통 `파일 시스템`을 통해 편하게 접근

</details>