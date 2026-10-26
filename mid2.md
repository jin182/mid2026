```c
#include <stdio.h>
#include <stdlib.h>
#include "sqlite3.h"

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

    rc = sqlite3_open("test.db", &db);
    if (rc != SQLITE_OK) {
        printf("Connection failed: %s\n", sqlite3_errmsg(db));
        return 1;
    }
    
    printf("SQLite3 connected successfully!\n\n");

    const char *sql_create = 
        "DROP TABLE IF EXISTS members;"
        "CREATE TABLE members ("
        "id INTEGER PRIMARY KEY AUTOINCREMENT,"
        "name TEXT NOT NULL,"
        "department TEXT NOT NULL);";

    rc = sqlite3_exec(db, sql_create, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        printf("Table creation failed: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }

    const char *sql_insert = 
        "INSERT INTO members (name, department) VALUES "
        "('Alice', 'DevTeam'),"
        "('Bob', 'PlanningTeam'),"
        "('Charlie', 'DevTeam'),"
        "('David', 'DesignTeam');";

    rc = sqlite3_exec(db, sql_insert, 0, 0, &err_msg);
    if (rc != SQLITE_OK) {
        printf("Data insertion failed: %s\n", err_msg);
        sqlite3_free(err_msg);
        sqlite3_close(db);
        return 1;
    }
    
    printf("Insertion completed (4 members).\n\n");

    printf("--- 'DevTeam' Search Results ---\n");
    const char *sql_search = "SELECT * FROM members WHERE department = 'DevTeam';";
    
    rc = sqlite3_exec(db, sql_search, callback, 0, &err_msg);
    if (rc != SQLITE_OK) {
        printf("Search query failed: %s\n", err_msg);
        sqlite3_free(err_msg);
    }
```
    sqlite3_close(db);
    return 0;
}
