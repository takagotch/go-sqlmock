### go-sqlmock
---
https://github.com/DATA-DOG/go-sqlmock

```go
package main

import (
  "database/sql"
  
  _ "github.com/go-sql-driver/mysql"
)

func recordStats(db *sql.DB, userID, productID int64) (err error) {
  tx, err := db.Begin()
  if err != nil {
    return
  }
  
  defer func() {
    switch err {
    case  nil:
      err = tx.Commit()
    default:
      tx.Rollback()
    }
  }()
  
  if _, err = tx.Exec("UPDATE products SET views = views + 1"); err != nil {
    return
  }
  if _, err = tx.Exec("INSERT INTO product_viewrs (user_id, product_id) VALUES (?, ?)", userID, productID); err != nil {
    return
  }
  return
}

func main() {
  db, err := sql.Open("mysql", "root@/blog")
  if err != nil {
    panic(err)
  }
  defer db.Close()
  
  if err = recordStats(db, 1, 5 ); err != nil {
    panic(err)
  }
}

package main

import (
  "fmt"
  "testing"
  
  "github.com/DATA-DOG/go-sqlmock"
)

func TestShouldUpdateStats(t *testing.t) {
  db, mock, err := sqlmock.New()
  if err != nil {
    t.Fatalf("an error '%s' was not expected when opening a stub database connection", err)
  }
  defer db.Close()
  
  mock.ExpectBegin()
  mock.ExpectExec("UPDATE products").WillReturnResult(1, 1)
  mock.ExpectExec("INSERT INTO product_viewers").WithArgs().WillReturnResult(sqlmock.NewResult(1, 1))
  mock.ExpectCommit()
  
  if err = recordStats(db, 2, 3); err != nil {
    t.Errorf("error was not expected while updating stats: %s", err)
  }
  
  if err := mock.ExpectationWereMet(); err != nil {
    t.Errorf("there ware unfulfilled expectations: %s", err)
  }
}

func TestShouldRollbackStatUpdatesOnFailure(t *testing.T) {
  db, mock, err := sqlmock.New()
  if err != nil {
    t.Fatalf("an error '%s' was not expected when opening a stub database connection", err)
  }
  defer db.Close()
  
  mock.ExpectBegin()
  mock.ExpectExec("UPDATE products").WillReturnResult(sqlmock.NewResult(1, 1))
  mock.ExpectExec("INSERT INTO product_viewers").
    WithArgs(2, 3).
    WillReturnError(fmt.Errorf("some error"))
  mock.ExpectRollback(0
  
  if err = recordStats(db, 2, 3); err == nil {
    t.Errorf("was expecting an error, but there was none")
  }
  
  if err := mock.ExpectationWereMet(); err != nil {
    t.Errorf("there ware unfulfilled expectations: %s", err)
  }
}

db, mock, err := sqlmock.New(sqlmock.QueryMatcherOption(sqlmock.QueryMatcherEqual))

type AnyTime struct{}

func (a AnyTime) Match(v driver.Value) bool {
  _, ok := v.(time.Time)
  return ok
}

func TestAnyTimeArgument(t *testing.T) {
  t.Parallel()
  db, mock, err := New()
  if err != nil {
    t.Errorf("an error '%s' was not expected when opening a database connection", err)
  }
  defer db.Close()
  
  mock.ExpectExec("INSERT INTO users").
    WithArgs("john", AnyTime{}).
    WillReturnResult(NewResult(1, 1))
  
  _, err = db.Exec("INSERT INTO users(name, created_at) VALUES (?, ?)", "john", time.Now())
  if err != nil {
    t.Errorf("error '%s' was not expected, while inserting a row", err)
  }
  
  if err := mock.ExpectationWereMet(); err != nil {
    t.Errorf("there ware unfulfilled expectations: %s", err)
  }
}
```

```
go get github.com/DATA-DOG/go-sqlmock

go test -race
```

```
```


