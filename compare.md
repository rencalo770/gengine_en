### start
- compare the performance between gengine and gopher_lua

### case
- function test is "get the value of the N-th term of fibonacci  sequence"
- gopher-lua github: https://github.com/yuin/gopher-lua 
- gengine github: https://github.com/bilibili/gengine


### machine resource
- System : MacOS
- CPU: 3.2 GHz Intel Core i7
- Memory: 16 GB 2667 MHz DDR4

### go.mod
```golang
go 1.15

require (
	github.com/bilibili/gengine v1.5.1
	github.com/yuin/gopher-lua v0.0.0-20200816102855-ee81675732da
)
```

### gopher-lua just lua
```golang

import (
	"fmt"
	lua "github.com/yuin/gopher-lua"
	"testing"
	"time"
)

const lua_fib = `
function fib(n)
	if n < 2 then
		return 1
	end
	return fib(n-1) + fib(n-2)
end
`

func Test_lua(t *testing.T) {

	L := lua.NewState()
	defer L.Close()
	// 加载fib.lua
	if err := L.DoString(lua_fib); err != nil {
		panic(err)
	}

	start := time.Now()
	defer logTime(start)
	// 调用fib(n)
	n := 20 //此处输入参数
	err := L.CallByParam(lua.P{
		Fn:      L.GetGlobal("fib"), // 获取fib函数引用
		NRet:    1,                  // 指定返回值数量
		Protect: true,               // 如果出现异常，是panic还是返回err
	}, lua.LNumber(n)) // 传递输入参数n=10
	if err != nil {
		panic(err)
	}
	// 获取返回结果
	ret := L.Get(-1)
	// 从堆栈中扔掉返回结果
	//L.Pop(1)
	// 打印结果
	res, ok := ret.(lua.LNumber)
	if ok {
		println(int(res))
	} else {
		fmt.Println("unexpected result")
	}
}

func logTime( t time.Time,) {
	println( "cost_time:", time.Since(t).Microseconds())
}

```

### gopher-lua assembly
```golang

import (
	lua "github.com/yuin/gopher-lua"
	"testing"
	"time"
)

func Test_inject(t *testing.T) {

	L := lua.NewState()
	defer L.Close()
	meta := L.NewTable()
	L.SetGlobal("Counter", meta)
	// 注册函数
	//L.SetField(meta, "new", L.NewFunction(newCounter))
	//L.SetField(meta, "incr", L.NewFunction(incrCounter))
	L.SetField(meta, "get", L.NewFunction(getCounter))
	L.SetField(meta, "fib", L.NewFunction(fibCounter))
	// 使用lua代码测试效果
	start := time.Now()
	defer logTime(start)
	err := L.DoString(`
		counter = Counter:fib(40)
		print(counter:get())
	`)
	if err != nil {
		panic(err)
	}
}

func fibCounter(L *lua.LState) int {
	c := L.NewTable()
	self := L.CheckTable(1)
	value := lua.LNumber(0)
	// 第二个为可选参数
	if L.GetTop() >= 2 {
		value = L.CheckNumber(2)
	}

	value = lua.LNumber(float64(fib(int(value))))
	L.SetField(c, "value", value)
	L.SetMetatable(c, self)
	L.SetField(self, "__index", self)
	// 返回值压栈
	L.Push(c)
	// 返回[函数返回值的个数]
	return 1
}


func fib(n int) int{
	if n < 2 {
		return 1
	}
	return fib(n-1) + fib(n-2)
}

func getCounter(L *lua.LState) int {
	self := L.CheckTable(1)
	value := L.GetField(self, "value").(lua.LNumber)
	// 返回值压栈
	L.Push(value)
	// 返回[函数返回值的个数]
	return 1
}

func logTime( t time.Time,) {
	println( "cost_time:", time.Since(t).Microseconds())
}

```

### gengine

```golang
import (
	"fmt"
	"github.com/bilibili/gengine/builder"
	"github.com/bilibili/gengine/context"
	"github.com/bilibili/gengine/engine"
	"testing"
	"time"
)

const gengine_fib = `
rule "1" "2" 
begin
r.R = fib(20) //此处输入参数
end
`

func Fib(n int) int{
	if n < 2 {
		return 1
	}
	return Fib(n-1) + Fib(n-2)
}


func Test_gengine(t *testing.T) {

	type Result struct{
		R int
	}

	r := &Result{}

	dataContext := context.NewDataContext()
	dataContext.Add("fib", Fib)
	dataContext.Add("r", r)
	dataContext.Add("println", fmt.Println)

	ruleBuilder := builder.NewRuleBuilder(dataContext)
	e1 := ruleBuilder.BuildRuleFromString(gengine_fib)
	if e1 != nil {
		panic(e1)
	}

	start := time.Now()
	defer logTime(start)

	gengine := engine.NewGengine()
	e := gengine.Execute(ruleBuilder, true)
	if e != nil {
		panic(e)
	}
	println(r.R)
}

func logTime( t time.Time,) {
	println( "cost_time:", time.Since(t).Microseconds())
}

```

### compare result

| get Nth value |gopher-lua lua cost time avg |gopher-lua assemble cost time avg| gengine cost time avg |
| -------- | -------- | --------- |----|
|n = 1 |0.008ms|0.15ms|0.045ms|
|n = 2 |0.01ms|0.15ms|0.045ms|
|n = 3 |0.012ms|0.15ms|0.045ms|
|n = 4 |0.015ms|0.15ms|0.045ms|
|n = 5 |0.02ms|0.15ms|0.045ms|
|n = 10 |0.033ms|0.15ms|0.045ms|
|n = 11 |0.05ms|0.15ms|0.045ms|
|n = 20 |2.9ms|0.2ms|0.08ms|
|n = 30 |365ms|4.3ms|4ms|
|n = 40 |43800ms |500ms|490ms|95|
|n = 50 |too long to get|61000ms|61000ms|-|

