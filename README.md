## sqlparser

Simply SQL and DDL parser for Go, this library is forked from [vitess-sqlparser](https://github.com/blastrain/vitess-sqlparser).

<br>

### Example of use

```go
package main

import (
	"encoding/json"
	"fmt"

	"github.com/zhufuyi/sqlparser/ast"
	"github.com/zhufuyi/sqlparser/parser"
)

func main() {
	var sqlData = `create table user (
    id         bigint unsigned auto_increment primary key comment 'user id',
    name       char(50)        not null comment 'username',
    email      char(50)        not null comment 'email',
    created_at datetime        default current_timestamp comment 'created time'
)
`

	type columnField struct {
		Name      string `json:"name"`
		Type      string `json:"type"`
		Comment   string `json:"comment"`
		IsPrimary bool   `json:"isPrimary"`
	}
	var columns []*columnField

	stmts, err := parser.New().Parse(sqlData, "", "")
	if err != nil {
		panic(err)
	}

	for _, stmt := range stmts {
		if ct, ok := stmt.(*ast.CreateTableStmt); ok {
			for _, col := range ct.Cols {
				filed := &columnField{
					Name: col.Name.String(),
					Type: col.Tp.String(),
				}
				for _, o := range col.Options {
					switch o.Tp {
					case ast.ColumnOptionPrimaryKey:
						filed.IsPrimary = true
					case ast.ColumnOptionComment:
						filed.Comment = o.Expr.GetDatum().GetString()
					}
				}
				columns = append(columns, filed)
			}
		}
	}

	jsonData, err := json.MarshalIndent(&columns, "", "  ")
	if err != nil {
		panic(err)
	}
	fmt.Println(string(jsonData))
}
```
