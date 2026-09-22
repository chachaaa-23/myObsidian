The ==`REAL` data type== 
: is an approximate number with floating point numeric data.

DB생성과 현재 사용중인 DB 변경, 테이블 생성 및 값 INSERT 를 동시에 하는 경우!
- 일괄 처리(batch) 오류 발생 가능
- **WHY:** 하나의 스크립트 블록, 동시에 컴파일 하려 할 때 명령 단위를 위에서부터 차례로 보지 않을 수 있음.
- **HOW TO**: `GO` 명령어 사용해서 명령단위 나누기. 
- 