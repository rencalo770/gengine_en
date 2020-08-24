# Extends Call in Rules
- golang's extends feature is not true extends, in fact it is composite mode, or you can say there is no extends in golang
- function implements some interfaces, delete the interface, there is no effect in golang.
- Mentioned earlier,in gengine, every call  as long as it does not exceed tow layer "A.B" form call, it is legal in gengine.

```go:

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
test: https://github.com/rencalo770/gengine/blob/master/test/complex/extends_test.go
