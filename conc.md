# Concurrent implementation within rules
- In many business scenarios, a rule requires a lot of data indicators to support, because of the amount of data, these data indicators are often impossible to store locally.
- Therefore, if many indicators are taken from the network sequentially within the rule, it will seriously slow down the execution performance of the rule.
- Based on this consideration, **gengine supports concurrent function calls or assignment statements in the rules**. Its syntax is quite simple, just use the conc() statement block to wrap the interfaces that require concurrent calls.

```go
const rule_conc_statement = `
rule "conc_test" "test"
begin
conc {// this should be used in time-consuming operation, such as the operation contains network connection (get data from remote based on network)
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
func Sout(str string) {
println("----",str)
}

//inject
dataContext.Add("println",fmt.Println)
dataContext.Add("sout", Sout)
```

Test: https://github.com/bilibili/gengine/blob/main/test/conc_statement_test.go
- Support placement methods, functions, and assignment statements in conc{} statement blocks
- The conc() statement block can be empty, or have one or more methods, functions or assignment statements
- When the conc() statement block is empty, it has no effect. When there is only one statement, it degenerates into sequential execution. When there are more than 2 or more statements, concurrent execution will be performed
- Users need to ensure that the api called in conc{} is thread-safe
- Perform a few more tests, you will find that the output order in conc() is different each time