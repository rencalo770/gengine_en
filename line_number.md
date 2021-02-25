# support line number
- gengine support line number when it runs error, it will help the user to find what went wrong quickly and easily

## example

 ```go

import (
	"fmt"
	"github.com/bilibili/gengine/builder"
	"github.com/bilibili/gengine/context"
	"github.com/bilibili/gengine/engine"
	"testing"
)

var lineNumberRules = `
rule "line_number"  "when execute error,gengine will give out error"
begin

//Println("golang", "hello", "world" )

//取消斜杠注释,依次测试不同的报错情况
//if Println("golang", "hello") == 100 {
// 	Println("golang", "hello")
//}

ms.X()

end
`

type MyStruct struct {

}

func (m *MyStruct)XX(s string)  {
	println("XX")
}


func Println(s1, s2 string) bool {
	println(s1, s2)
	return false
}


func Test_number(t *testing.T) {
	dataContext := context.NewDataContext()
	//注入自定义函数
	dataContext.Add("Println", Println)
	ms :=  &MyStruct{}
	dataContext.Add("ms", ms)

	ruleBuilder := builder.NewRuleBuilder(dataContext)
	e1 := ruleBuilder.BuildRuleFromString(lineNumberRules)
	if e1 != nil {
		panic(e1)
	}

	eng := engine.NewGengine()
	// true: means when there are many rules, if one rule execute error,continue to execute rules after the occur error rule
	e2 := eng.Execute(ruleBuilder, true)
	if e2 != nil {
		println(fmt.Sprintf("%+v", e2))
	}
}
``` 

- test code: https://github.com/bilibili/gengine/blob/main/test/line_number/line_number_test.go
- test result:
```go
=== RUN   Test_number
[rule: "line_number" executed, error:
 line 12, column 0, code: ms.X(), NOT FOUND Function: X ]
--- PASS: Test_number (0.00s)
PASS

```
