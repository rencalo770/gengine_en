# Based on StopTag to Control to Stop Rules

### Backgroud
in some scene, user need this support: <br/>
- in sort model, it has rules named "1","2","3","4","5","6", when user execute rule "2" finished, and don't want to execute last rules.<br/>
- in mix model, it has rules named "1","2","3","4","5","6", and the rule "1" has the highest priority,when user execute rule "1" finished, and don't want to execute last rules.<br/>
- for this consider, or for performance,**gengine support use "StopTag" to meet the user needed**.

### Example

#### Sort model Scene
```go
func (g *Gengine) ExecuteWithStopTagDirect(rb *builder.RuleBuilder, b bool, sTag *Stag) error
```
- test case: https://github.com/rencalo770/gengine/blob/master/test/stop_tag_test/stop_tag_in_sort_model_test.go

support in gengine pool:

```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error 

//and
func (gp *GenginePool)ExecuteRulesWithMultiInputAndStopTag(data map[string]interface{}, stag *Stag) error
```

#### Mix Model Scene
```go 
func (g *Gengine) ExecuteMixModelWithStopTagDirect(rb * builder.RuleBuilder, sTag *Stag)
```

- test case: https://github.com/rencalo770/gengine/blob/master/test/stop_tag_test/stop_tag_in_mix_model_test.go
support in gengine pool:

```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error 

//and
func (gp *GenginePool)ExecuteRulesWithMultiInputAndStopTag(data map[string]interface{}, stag *Stag) error
```
- ATTENTION: in gengine pool,use the same function to control,because the control of execute model has been split to```func (gp *GenginePool)SetExecModel(execModel int) error```