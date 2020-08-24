# Exceute the Rules Based on User selected
### Background
We saw this demand when user use gengine:<br/>
- Suppose there are 10 rules in gengine instance,rules' names tagged from "1" to "10", when a request comes in, user knows what rules he wants to execute .
- **gengine support user to select rules to execute by specifying the rules' name**, it will more suitable for user, and it will be high performance.


### Method

#### ExecuteSelectedRules
```go
func (g *Gengine)ExecuteSelectedRules(rb * builder.RuleBuilder, names []string)
```
- parameter "names", translate rules' names to this function, such as when```name :=[]string{"1", "7", "3"}```,gengine execute the 3 user-selected rules based on rules' priority.  
- if user selected rules don't exist, gengine will log it.

#### ExecuteSelectedRulesConcurrent
```go
func (g *Gengine)ExecuteSelectedRulesConcurrent(rb * builder.RuleBuilder, names []string) 
```
- parameter "names", translate rules' names to this function, such as when```name :=[]string{"1", "7", "3"}```,gengine execute the 3 user-selected rules concurrently.  
- if user selected rules don't exist, gengine will log it.

### Implements
- https://github.com/rencalo770/gengine/blob/master/engine/gengine.go

### Test
- https://github.com/rencalo770/gengine/blob/master/test/exceute_selected_rules_test.go
