# Framework of Gengine 

- there ara only four key APIs(files), they are ```dataContext```,```ruleBuilder```,```engine``` and ```GenginePool```

### dataContext
- Code : https://github.com/bilibili/gengine/blob/main/context/data_context.go
- ```dataContext``` allows user to inject APIs that will be used in gengine. 
- example:
```golang
	dataContext := context.NewDataContext()
	//inject API
	dataContext.Add("println",fmt.Println)
```


### ruleBuilder
- Code: https://github.com/bilibili/gengine/blob/main/builder/rule_builder.go
- ruleBuilder receive ```dataContext``` instance as a parameter, and compile the string the user input to build executable rule code
- example:
```go
const rule = `
rule "1" "rule-des" salience 10
begin
println("hello world, gengine!")
end
`

	dataContext := context.NewDataContext()
	//inject API
	dataContext.Add("println",fmt.Println)
	
	ruleBuilder := builder.NewRuleBuilder(dataContext)
	err := ruleBuilder.BuildRuleFromString(rule)
```


### engine
- Code: https://github.com/bilibili/gengine/blob/main/engine/gengine.go
- ```engine``` receive ```ruleBuilder``` instance as a parameter, and choose a execute model the user need to execute the rules in gengine
- example:
```go
const rule = `
rule "1" "rule-des" salience 10
begin
println("hello world, gengine!")
end
`

	dataContext := context.NewDataContext()
	//inject API
	dataContext.Add("println",fmt.Println)
	
	ruleBuilder := builder.NewRuleBuilder(dataContext)
	err := ruleBuilder.BuildRuleFromString(rule)
	
	eng := engine.NewGengine()
	err := eng.Execute(ruleBuilder, true)
```

### GenginePool
- Code: https://github.com/bilibili/gengine/blob/main/engine/gengine_pool.go
- the APIs(and names) between gengine and gengine pool are one to one correspondence！
- gengine pool,support user to use gengine in high QPS,it solve the high QPS and thread-safety questions. it was illustrated in the other section in this doc.



  
  
  


