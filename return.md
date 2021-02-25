### return grammar
- gengine support return grammar

### in single instance
- test framework code
```go
import (
	"github.com/bilibili/gengine/builder"
	"github.com/bilibili/gengine/context"
	"github.com/bilibili/gengine/engine"
	"testing"
)

//rules, map[string]interface{} key is rule name, interface{} is return value
func framework(rules string) map[string]interface{} {
	dataContext := context.NewDataContext()
	dataContext.Add("getInt", getInt)
	ruleBuilder := builder.NewRuleBuilder(dataContext)
	e1 := ruleBuilder.BuildRuleFromString(rules)
	if e1 != nil {
		panic(e1)
	}

	eng := engine.NewGengine()
	e2 := eng.Execute(ruleBuilder, true)
	if e2 != nil {
		println(e2)
	}

	resultMap, _ := eng.GetRulesResultMap()
	return resultMap
}

func getInt() int {
	return 666
}

```

- the all below test code is based on the above framework code
- test code: https://github.com/bilibili/gengine/blob/main/test/return_statement_test.go

1. return nil
```go

func Test_return_nil_1(t *testing.T) {
	//无返回值,返回nil
	ruleName := "return_in_statements"
	rule := `rule  "` + ruleName + `"
			 begin
			  return // return nothing
      		 end	
			`
	returnResultMap := framework(rule)

	r := returnResultMap[ruleName]
	if r == nil {
		println("return is nil")
	}
}
```
 
2. return anything you want
```go
//this is different from golang do
func Test_return_complex_if_return_int64(t *testing.T)  {

	ruleName := `return_in_statements`
	rule := `rule  "` + ruleName + `"
			 begin
			  a = 290	//return different value as you change 
			  if a < 100 {
				  return 5 + 6 * 6			
    		  }else if a >= 100 && a < 200  {
  				  return "hello world"	
              }else {
				  return true	
			  }
      		 end	
			`
	returnResultMap := framework(rule)
	r := returnResultMap[ruleName]

	// a < 100时 return int64
	//i := r.(int64)

	//a >= 100 && a < 200时 return string
	//i := r.(string)

	//a >= 200
	i := r.(bool)

	println("return--->", i)
}
```

3. Attention: return statement should not be in conc{} grammar

### return in gengine pool 
- test code: https://github.com/bilibili/gengine/blob/main/test/pool_return_statements_test.go

```go
import (
	"fmt"
	"github.com/bilibili/gengine/engine"
	"testing"
	"time"
)

func random() int {
	return time.Now().Nanosecond()
}


func Test_pool_return_statments(t *testing.T) {

	ruleName := "test_pool_return"
	rule := `rule "` +ruleName +  `"  
			begin
				return random()
			end
			`

	apis := make(map[string]interface{})
	apis["print"] = fmt.Println
	apis["random"] = random
 
	pool, e1 := engine.NewGenginePool(1, 2, 1, rule, apis)
	if e1 != nil {
		println(fmt.Sprintf("e1: %+v", e1))
	}

  
	data := make(map[string]interface{})
	e2, rrm1 := pool.ExecuteRulesWithMultiInputWithSpecifiedEM(data)
	if e2 != nil {
		panic(e2)
	}

	i1 := rrm1[ruleName]
	ix1 := i1.(int)
	println("ix1--->", ix1)

	e3, rrm2 := pool.ExecuteRulesWithMultiInputWithSpecifiedEM(data)
	if e3 != nil {
		panic(e2)
	}

	i2 := rrm2[ruleName]
	ix2 := i2.(int)
	println("ix2--->", ix2)

	i3 := rrm1[ruleName]
	ix3 := i3.(int)
	println("ix3--->", ix3)
}
```
 
