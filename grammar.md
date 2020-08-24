# Gengine Grammar

### gengine DSL Grammar
```go
const rule = `
rule "rulename" "rule-description" salience  10
begin

//rule body

end`
```
as bellow, gengine DSL is composed of:
- keyword rule, it is followed closely by "rule name" and "rule description",both of them are needed. when ****there are many rules in one gengine instance, rule's name must be unique****, or the rules with same name will be left just one!
- keyword salience, is followed by an integer ,it means the priority of the rule, it is need, but not must be needed. when user don't set it, it will be unknown!
- keyword begin and keyword end wrap the rule body(rule logic),they are must be needed!

### Rule Body Grammar

 the grammar the gengine support is similar to golang, java, C/C++ and so on， but it is more easy!
 
#### Supported Calculate of  Rule Body:
- Support addition, subtraction, multiplication and division operations, and addition between string
- Complete logic operation(&&、 ||、 !)
- Operator: ==, !=, \>, <,  \>=, <=
- support brackets 
- priority decent one by one: brackets, !, multiplication and division, addition and subtraction, logic(&&,||) 

#### Support Base Type of Rule Body
- string
- bool
- int, int8, int16, int32, int 64
- uint, uint8, uint16,uint32, uint64
- float32, float64

#### NOT support
- not support directly to handle nil, you can define a variable to accept nil in rule, and define a function to handle it! 

#### Rule Body Supports the Grammar 
- if .. else if .. else , or it nested structure

```go
const rule_else_if_test =`
rule "elseif_test" "test"
begin

a = 8
if a < 1 {
	println("a < 1")
} else          if a >= 1 && a <6 {
	println("a >= 1 && a <6")
} else if a >= 6 && a < 7 {
	println("a >= 6 && a < 7")
} else if a >= 7 && a < 10 {
	println("a >=7 && a < 10")
} else        {
	println("a > 10")
}
end`
```

- conc{} grammar block

```go
const rule_conc_statement  = `
rule "conc_test" "test" 
begin
	conc  { 
		a = 3.0
		b = 4
		c = 6.8
		d = "hello world"
        e = "you will be happy here!"
	}
	println(a, b, c, d, e)
end`
```


#### @name
- use @name in rule, @name will be explain to the current rule name string which @name in
- test: https://github.com/rencalo770/gengine/blob/master/test/at_name_test.go

#### Support Comment
- support single line comment in rules begins // 

#### User-defined Variable 
- when user define a variable, don't need to declare the type 
- ****a variable defined in rule is only visible to itself****(local variable)
- ****the variable injected by using dataContext, it is visible to all rule in gengine****(global variable)

#### Other Grammar 

- for simplify to use and  remember the rule just support the longest call form is  "A.B", not support longer call form such as "A.B.C"(Dimit's Law)
- for grantee make the rule grammar is not related to golang' grammar, in rule, you can call multi-return function or method, but when you define a variable to receive the return, rule just support single-return function or method(it is suggest user to set default value for return, not to return error)


