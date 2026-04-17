# feat/database-schema — DB 스키마 + 시드 + 차량/대시보드 API

## 포함 파일

**백엔드**
- `backend/src/main/java/com/evernex/bms/db/SchemaInitializer.java` — `CREATE TABLE ...`
- `backend/src/main/java/com/evernex/bms/db/DataSeeder.java` — 초기 공장/차량/관리자 시드
- `backend/src/main/java/com/evernex/bms/controller/VehiclesController.java` — GET /vehicles, /vehicles/{id}
- `backend/src/main/java/com/evernex/bms/controller/DashboardController.java` — GET /dashboard/*

**SQL 스크립트**
- `sql/` — 스키마 DDL, 테스트 시드, 마이그레이션

## 의존 브랜치
- `feat/project-setup`

## 적용
같은 상대 경로로 복사 후 커밋.
