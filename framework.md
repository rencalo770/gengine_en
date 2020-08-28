# Framework of Gengine 


- from version v1.2.0, we simplified the framework of gengine, users just need make a little change to  move from old version to new version.
- from version v1.2.0, user just need to pay attention to 4 important API of gengine, they are ```dataContext```, ```ruleBuilder``` , ```engine``` and ```GenginePool```
- from version v1.2.0, the code of gengine is more easy to understand and use, illustration as follow:

### dataContext
- Code : https://github.com/rencalo770/gengine/blob/master/context/data_context.go
- ```dataContext``` allows user to inject APIs that will be used in gengine. 
- example:
```golang
	dataContext := context.NewDataContext()
	//inject API
	dataContext.Add("println",fmt.Println)
```


### ruleBuilder
- Code: https://github.com/rencalo770/gengine/tree/master/builder
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
- Code: https://github.com/rencalo770/gengine/blob/master/engine/gengine.go
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
- gengine pool,support user to use gengine in high QPS,it solve the high QPS and thread-safety questions. it was illustrated in the other section in this doc.

### internal
- internal dir contains the code code of gengine, use need not to care about it.



  
  
  


