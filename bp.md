# Best Practices
- practise in many business, we find that the hardest questions when use gengine is that concurrent questions. So made tow examples for you in real business.

### example 1
- do not use gengine pool in business.(we still recommend you to use gengine pool)

```golang
package server

import (
	"errors"
	"fmt"
	"gengine/builder"
	"gengine/context"
	"gengine/engine"
	"testing"
)

//不使用gengine pool,但是也要实现多gengine实例
type IEngine struct {
	Rb       *builder.RuleBuilder
	Engine   *engine.Gengine
	IVersion int
}

type MySelfService struct {
	IEngineChan chan *IEngine
	RbChan      chan *builder.RuleBuilder

	//规则版本更新控制
	ApiOuter map[string]interface{}
	Version  int
	Len      int

	//other params
}

//请确保注入的API是线程安全的
func NewService(iNum int, ruleStr string, apiOuter map[string]interface{}) *MySelfService {
	if iNum < 1 {
		panic(fmt.Sprintf("engines' number should be bigger than 0!"))
	}

	//这个长度，使用者自己看着写
	if len(ruleStr) < 1 {
		panic(fmt.Sprintf("rules len is 0"))
	}

	enginesChan := make(chan *IEngine, iNum)
	for i := 0; i < iNum; i++ {
		enginesChan <- &IEngine{
			Rb:       nil,
			Engine:   engine.NewGengine(),
			IVersion: 0,
		}
	}

	buildersChan := make(chan *builder.RuleBuilder, iNum)
	for i := 0; i < iNum; i++ {
		rb, e := makeRuleBuilder(ruleStr, apiOuter)
		if e != nil {
			panic(fmt.Sprintf("build rule err:%+v", e))
		}
		buildersChan <- rb
	}

	return &MySelfService{
		IEngineChan: enginesChan,
		RbChan:      buildersChan,
		ApiOuter:    apiOuter,
		Version:     1, //初始化时要和IVersion不同
		Len:         iNum,
	}
}

//this could ensure make thread safety!
func makeRuleBuilder(ruleStr string, apiOuter map[string]interface{}) (*builder.RuleBuilder, error) {
	dataContext := context.NewDataContext()
	if apiOuter != nil {
		for k, v := range apiOuter {
			dataContext.Add(k, v)
		}
	}

	rb := builder.NewRuleBuilder(dataContext)
	if ruleStr != "" {
		if e := rb.BuildRuleFromString(ruleStr); e != nil {
			rb.Kc.ClearRules()
			return nil, errors.New(fmt.Sprintf("build rule from string err: %+v", e))
		}
	} else {
		return nil, errors.New("the ruleStr is \"\"")
	}
	return rb, nil
}

//异步构建规则，可确保构建规则不影响程序性能，美滋滋
func (ms *MySelfService) UpdateRules(ruleStr string) error {
	buildersChan := make(chan *builder.RuleBuilder, ms.Len)
	for i := 0; i < ms.Len; i++ {
		rb, e := makeRuleBuilder(ruleStr, ms.ApiOuter)
		//更新规则
		if e != nil {
			return e
		}
		buildersChan <- rb
	}
	//先更新好,最后才能更新版本号
	ms.RbChan = buildersChan
	ms.Version++
	return nil
}

func (ms *MySelfService) Service(req *Request) (*Response, error) {
	iEngine := <-ms.IEngineChan
	defer func() {
		ms.IEngineChan <- iEngine
	}()

	//版本号不同说明要更新ruleBuilder
	if iEngine.IVersion != ms.Version {
		//同步更新规则,同时同步版本号
		iEngine.Rb = <-ms.RbChan
		iEngine.IVersion = ms.Version
	}

	//inject additional api
	iEngine.Rb.Dc.Add("req", req)
	resp := &Response{}
	iEngine.Rb.Dc.Add("resp", resp)

	e := iEngine.Engine.ExecuteSelectedRules(iEngine.Rb, []string{"1", "2"})
	if e != nil {
		return nil, e
	}
	return resp, nil
}

//模拟使用
func Test_self(t *testing.T) {
	apis := make(map[string]interface{})
	apis["println"] = fmt.Println
	apis["room"] = &Room{}
	msr := NewService(10, service_rules, apis)

	//调用
	req := &Request{
		Rid:       123,
		RuleNames: []string{"1", "2"},
	}
	response, e := msr.Service(req)
	if e != nil {
		println(fmt.Sprintf("service err:%+v", e))
		return
	}
	println("resp result = ", response.At, response.Num)
}
```

### example 2
- use gengine pool


```golang

package server

import (
	"fmt"
	"gengine/engine"
	"testing"
)

//业务规则
const service_rules string = `
rule "1" "1"
begin
	resp.At = room.GetAttention()
	println("rule 1...")
end 

rule "2" "2"
begin
	resp.Num = room.GetNum()
	println("rule 2...")
end
`

//业务接口
type MyService struct {
	//gengine pool
	Pool *engine.GenginePool

	//other params
}

//request
type Request struct {
	Rid       int64
	RuleNames []string
	//other params
}

//resp
type Response struct {
	At  int64
	Num int64
	//other params
}

//特定的场景服务
type Room struct {
}

func (r *Room) GetAttention( /*params*/ ) int64 {
	// logic
	return 100
}

func (r *Room) GetNum( /*params*/ ) int64 {
	//logic
	return 111
}

//初始化业务服务
func NewMyService(poolMinLen, poolMaxLen int64, em int, rulesStr string, apiOuter map[string]interface{}) *MyService {
	pool, e := engine.NewGenginePool(poolMinLen, poolMaxLen, em, rulesStr, apiOuter)
	if e != nil {
		panic(fmt.Sprintf("初始化gengine失败，err:%+v", e))
	}

	myService := &MyService{Pool: pool}
	return myService
}

//service
func (ms *MyService) Service(req *Request) (*Response, error) {

	resp := &Response{}

	//基于需要注入接口或数据
	data := make(map[string]interface{})
	data["req"] = req
	data["resp"] = resp

	//模块化业务逻辑,api
	room := &Room{}
	data["room"] = room

	e := ms.Pool.ExecuteSelectedRulesWithMultiInput(data, req.RuleNames)
	if e != nil {
		println(fmt.Sprintf("pool execute rules error: %+v", e))
		return nil, e
	}

	return resp, nil
}

//模拟调用
func Test_run(t *testing.T) {

	//初始化
	//注入api，请确保注入的API属于并发安全
	apis := make(map[string]interface{})
	apis["println"] = fmt.Println
	msr := NewMyService(10, 20, 1, service_rules, apis)


	//调用
	req := &Request{
		Rid:       123,
		RuleNames: []string{"1", "2"},
	}
	response, e := msr.Service(req)
	if e != nil {
		println(fmt.Sprintf("service err:%+v", e))
		return
	}

	println("resp result = ", response.At, response.Num)
}

```





 

  