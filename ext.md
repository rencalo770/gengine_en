# Inheritance calls within rules
- Golang's inheritance and implementation are not as straightforward as java. Golang's inheritance is actually a combination mode, or it can be said that golang does not inherit the concept
- The method implements an interface. When the interface definition is deleted, it has no effect on the compilation and execution of golang code.
- As mentioned earlier, in gengine, as long as it does not exceed the two-layer call writing in the form of A and B, it is a legal syntax.

```go

//define
type Man struct {
Eat string
Drink string
}

// define
type Father struct {
*Man
Son string
}

//rule
const ext_rule = `
rule "extends test" "extends test"
begin
Father.Son = "tom"
Sout(Father.Son)
Father.Eat= "apple"
Sout(Father.Eat)
end
`
```
Test: https://github.com/bilibili/gengine/blob/main/test/complex/extends_test.go