```c
#include <stdio.h>
#include <stdlib.h>
#include "sqlite3.h"

// SELECT 쿼리 결과를 출력하기 위한 콜백 함수
static int callback(void *NotUsed, int argc, char **argv, char **azColName) {
    for (int i = 0; i < argc; i++) {
        printf("%s: %s | ", azColName[i], argv[i] ? argv[i] : "NULL");
    }
    printf("\n");
    return 0;
}

int main(void) {
    sqlite3 *db = NULL;
    char *err_msg = 0;
    int rc;

    // 1. 데이터베이스 연결
    rc = sqlite3_open("test.db", &db);
    if (rc != SQLITE_OK) {
        printf("연결 실패: %s\n", sqlite3_errmsg(db));
        return 1;
    }
    printf("SQLite3 연결 성공!\n\n");

    // 2. 테이블 설계 및 생성 (기존 테이블 초기화 포함)
    const char *sql_create = 
        "DROP TABLE IF EXISTS members;"
        "CREATE TABLE members ("
        "id INTEGER PRIMARY KEY AUTOINCREMENT,"
        "name TEXT NOT NULL,"
        "department TEXT NOT NULL);";

    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        printf("테이블 생성 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    // 3. 3명 이상의 데이터 삽입[cite: 1]
    const char *sql_insert = 
        "INSERT INTO members (name, department) VALUES "
        "('김철수', '개발팀'),"
        "('이영희', '기획팀'),"
        "('박민수', '개발팀'),"
        "('최지윤', '디자인팀');";

    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        printf("데이터 삽입 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }
    printf("4명의 데이터 삽입을 완료했습니다.\n\n");

    // 4. 검색 기능 구현 (시나리오: '개발팀' 소속 검색)[cite: 1]
    printf("--- '개발팀' 검색 결과 ---\n");
    const char *sql_search = "SELECT * FROM members WHERE department = '개발팀';";
    
    rc = sqlite3_exec(db, sql_search, callback, 0, &err_msg);
    if (rc != SQLITE_OK) {
        printf("검색 쿼리 실패: %s\n", err_msg);
        sqlite3_free(err_msg);
    }

    // 5. 데이터베이스 연결 종료
    sqlite3_close(db);
    return 0;
}`
```
