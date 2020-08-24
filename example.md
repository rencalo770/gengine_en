# gengine example
there is an easy example of gengine 

#### example
```go

package test

import (
	"bytes"
	"fmt"
	"gengine/base"
	"gengine/builder"
	"gengine/context"
	"gengine/engine"
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
	knowledgeContext := base.NewKnowledgeContext()
	ruleBuilder := builder.NewRuleBuilder(knowledgeContext,dataContext)


	start1 := time.Now().UnixNano()
    //build rule
	err := ruleBuilder.BuildRuleFromString(rule1) //string(bs)
	end1 := time.Now().UnixNano()

	logrus.Infof("rules num:%d, load rules cost time:%d", len(knowledgeContext.RuleEntities), end1-start1 )

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
- from the example, we can find that ,****the compile of rules and the execute of rule is async****. so user can use this feature, to update the rules in gengine but not need to stop service.
- Usually, create a new ```ruleBuilder```to accept new rules and compile, when compile finished, then use the new ```ruleBuilder```pointer to replace old ```ruleBuilder```pointer to update the rules in gengine.
 in addtion,user can use ```ruleBuilder``` to check the grammar of rules in async.
- ATTENTION:  compile rules is CPU INTENSIVE，commonly, when user update the rules and there are many gengine instances in your service，please just build a ```ruleBuilder```, the last gengine instance get the updated rules by copy the new ```ruleBuilder```. 




