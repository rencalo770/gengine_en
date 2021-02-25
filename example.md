# gengine example
There is an easy example of gengine, in real business, we highly recommend you to use gengine pool API (https://github.com/bilibili/gengine/blob/main/engine/gengine_pool.go) 

#### example
```go

package test

import (
	"bytes"
	"fmt"
	"github.com/bilibili/gengine/builder"
	"github.com/bilibili/gengine/context"
	"github.com/bilibili/gengine/engine"
	"github.com/sirupsen/logrus"
	"io/ioutil"
	"strconv"
	"strings"
	"testing"
	"time"
)

//struct need to be inject
type User struct {
	Name string
	Age  int64
	Male bool
}

func (u *User)GetNum(i int64) int64 {
	return i
}

func (u *User)Print(s string){
	fmt.Println(s)
}

func (u *User)Say(){
	fmt.Println("hello world")
}

//define a rule
const rule1 = `
rule "name test" "i can"  salience 0
begin
		if 7 == User.GetNum(7){
			User.Age = User.GetNum(89767) + 10000000
			User.Print("6666")
		}else{
			User.Name = "yyyy"
		}
end
`
func Test_Multi(t *testing.T){
	user := &User{
		Name: "Calo",
		Age:  0,
		Male: true,
	}

	dataContext := context.NewDataContext()
	//inject struct
    dataContext.Add("User", user)

	//init rule engine
	ruleBuilder := builder.NewRuleBuilder(dataContext)


	start1 := time.Now().UnixNano()
    //build rule
	err := ruleBuilder.BuildRuleFromString(rule1) //string(bs)
	end1 := time.Now().UnixNano()

	logrus.Infof("rules num:%d, load rules cost time:%d", len(ruleBuilder.Kc.RuleEntities), end1-start1 )

	if err != nil{
		logrus.Errorf("err:%s ", err)
	}else{
		eng := engine.NewGengine()

		start := time.Now().UnixNano()
        //execute rules
		err := eng.Execute(ruleBuilder,true)
		println(user.Age)
		end := time.Now().UnixNano()
		if err != nil{
			logrus.Errorf("execute rule error: %v", err)
		}
		logrus.Infof("execute rule cost %d ns",end-start)
		logrus.Infof("user.Age=%d,Name=%s,Male=%t", user.Age, user.Name, user.Male)
	}
}
```

#### Explain
- User is a struct need to be inject into gengine；it should be init before injection；struct should be inject in pointer form, or you will not change the field value of struct!
- rule1 is Specific rules defined in string.
- dataContext is used to accept struct, function, method or other data which will be used in gengine
- ruleBuilder is used to compile rules which are in string form
- engine accept ruleBuilder, and to execute rules with the execute-model which user chooses

#### Tips
- From the example, we can find that ,****the compile of rules and the execute of rule is async****. so user can use this feature, to update the rules in gengine but not need to stop service.
- ATTENTION: compile rules is CPU INTENSIVE, We highly recommend you to complete your business by gengine pool API(https://github.com/bilibili/gengine/blob/main/engine/gengine_pool.go)
- In addtion,user can use ```ruleBuilder``` to check the grammar of rules in async.