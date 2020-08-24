# Concurrent in rule
 in many scene, a rule need many data metrics,because of the amount of data, it will not be store in local storage.
- so, if you get these data metrics one by one from network, it will make execute rules low performance!
- Consider this, **gengine support concurrent in rules' body**. grammar is easy , use conc{} to wrap the expressions what you concurrently execute.

```go
const rule_conc_statement  = `
rule "conc_test" "test" 
begin
	conc  { // this should be used in time-consuming operation, such as the operation contains network connection (get data from remote based on network) 
		println("hihihi")
		a = 3.0
		b = 4
		c = 6.8
		d = "hello world"
        e = "you will be happy here!"
		sout("heheheheh")
	}
	println(a, b, c, d, e)
end`

//define
func Sout(str string)  {
	println("----",str)
}

//inject
dataContext.Add("println",fmt.Println)
dataContext.Add("sout", Sout)
```

test: https://github.com/rencalo770/gengine/blob/master/test/conc_statement_test.go
- in conc{}, it support method, function and expressions
- conc{} can be null, or multiple method,function and expression statements 
- if conc{} is null, it can make no effect,when there is only one statement,it will sort to execute, when there more than 2 statements, it will execute concurrently  
- user must grantee that API is thread-safety which called in  conc{}













