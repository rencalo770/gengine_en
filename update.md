# update rules or remove rules
- it is also named how to compile rules
 

### Full update

```go
//in gengine
ruleBuilder := builder.NewRuleBuilder(dataContext)
e1 := ruleBuilder.BuildRuleFromString(rulesString) //这里全量更新

//in gengine pool
//first, when pool init, is full update
//second is
func (gp *GenginePool) UpdatePooledRules(ruleStr string) error

```

### Incremental update

``` 
//in gengine rule_builder.go
func (builder *RuleBuilder)BuildRuleWithIncremental(ruleString string) error

//in gengine pool
func (gp *GenginePool) UpdatePooledRulesIncremental(ruleStr string) error 
```

- test code : https://github.com/bilibili/gengine/blob/main/test/increment_update_test.go 


### remove rule
- batch delete

```
//ruleBuilder中
func (builder *RuleBuilder) RemoveRules(ruleNames []string) error 

//pool
func (gp *GenginePool) RemoveRules(ruleNames []string) error 

```
