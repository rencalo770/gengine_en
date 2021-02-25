# Based on the flag bit to control whether the rule continues to execute

### Background
Certain business scenarios have this requirement: <br/>
- For example, in a scenario where the rules are executed in sequential mode, there are rules "1", "2", "3", "4", "5", "6". When the user executes the rule "2", he does not want to continue execution The rules behind.<br/>
- For example, in a scenario where the mixed mode is used to execute the rules, there are rules "1", "2", "3", "4", "5", "6", and the rule "1" has the highest priority and the rest The priority of the rules is lower than "1". After the user executes "1", they don't want to continue to execute the following rules.<br/>
- In response to the above requirements, or based on performance considerations, **gengine supports users to use StopTag to meet the needs of the above two scenarios**.

### Example

#### Sequence mode scene
```go
func (g *Gengine) ExecuteWithStopTagDirect(rb *builder.RuleBuilder, b bool, sTag *Stag) error
```
-Test case: https://github.com/bilibili/gengine/blob/main/test/stop_tag_test/stop_tag_in_sort_model_test.go


#### Mixed mode scene
```go
func (g *Gengine) ExecuteMixModelWithStopTagDirect(rb * builder.RuleBuilder, sTag *Stag)
```

-Test case: https://github.com/bilibili/gengine/blob/main/test/stop_tag_test/stop_tag_in_mix_model_test.go