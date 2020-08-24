# Gengine Pool
"rule engine pool", it means there are  many instances in a pool,"rule engine pool" is the same as "database connection pool", both of them wants to solve the performance and thread-safety questions.

### Background

- all rules loaded to gengine, when executing, they are stateful.
- when just once gengine instance in a service, and a little rules in gengine, a request data can exceute rules very quickly, and the state will be keep very shortly, so it will not cause any accident;
- when just once gengine instance in a service, and many rules in gengine, especially high QPS, a request is not be finish, but another has come, in this condition, state will be break, and it will lead to unpredictable results.
- Consider this, gengine support gengine pool.

### Code
https://github.com/rencalo770/gengine/blob/master/engine/gengine_pool.go

### Gengine Pool Init
```go
//init a pool
func NewGenginePool(poolMinLen ,poolMaxLen int64, em int, rulesStr string, apiOuter map[string]interface{})
```
#### Parameters
- parameter "poolMinLen", the min number of gengine instance in pool 
- parameter "poolMaxLen", the min number of gengine instance in pool, and poolMaxLen > poolMinLen; when there is not enough gengine instance to execute, you can init additional the number poolMaxLen-poolMinLen gengine instances.
- parameter "em", use to set the execute model, em must be 1、2、3; when em=1, sort execute; when em=2, concurrent model; when em=3, mix model;when use ```ExecuteSelectedRulesWithMultiInput``` and ```ExecuteSelectedRulesConcurrentWithMultiInput```, this parameter will be invalid.
- parameter "rulesStr", the rules string
- parameter "apiOuter", the APIs user want to inject into gengine

### Dynamic Update Rules
```go 
//update the rules in gengine
func (gp *GenginePool)UpdatePooledRules(ruleStr string) error
```
the function allow user to update rules in gengine, when pool is in service, and it is thread safety.

### Clear Rules
```go
//clear all rules in gengine
func (gp *GenginePool)ClearPoolRules()
```
this function allow user to clear all rules when pool is in service, and it is thread safety.

### Set Execute Model
```go
//set execute model
func (gp *GenginePool)SetExecModel(execModel int) error 
```
this function allow user to change the execute model when pool is in service, and it is thread safety.
- execModel must be 1, 2 or 3
- when use```ExecuteSelectedRulesWithMultiInput```和```ExecuteSelectedRulesConcurrentWithMultiInput```, this method will be invalid.

### Excuet Rules

#### ExecuteRules
 ```go
func (gp *GenginePool)ExecuteRules(reqName string, req interface{}, respName string, resp interface{}) error
```
- reqName, the name of req which in gengine
- req, user request data
- respName, the name of resp which in gengine
- resp,response data
- use it, you only can transfer 2 parameters req and resp. 

#### ExecuteRulesWithMultiInput
```go
func  (gp *GenginePool)ExecuteRulesWithMultiInput(data map[string]interface{}) error
```
- ```ExecuteRules``` allow user to transfer any more parameter the user wanted to.

#### ExecuteRulesWithStopTag
```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error
```
- the difference between ```ExecuteRules``` and this function  is that it allow user to control whether continue to execute last rules.

#### ExecuteRulesWithMultiInputAndStopTag
```go
func (gp *GenginePool)ExecuteRulesWithMultiInputAndStopTag(data map[string]interface{}, stag *Stag) error 
```
- this function extends ```ExecuteRulesWithMultiInput```, and allow user to transfer any more parameter the user wanted to.
- and  extends ```ExecuteRulesWithStopTag``` and that it allow user to control whether continue to execute last rules.

#### ExecuteSelectedRulesWithMultiInput
```go
func  (gp *GenginePool)ExecuteSelectedRulesWithMultiInput(data map[string]interface{}, names []string) error
```
- this function allow user to transfer any more parameter the user wanted to.
- this function allow user to choose rules to run.
- this function will execute rules in sort model.

#### ExecuteSelectedRulesConcurrentWithMultiInput
```go
func  (gp *GenginePool)ExecuteSelectedRulesConcurrentWithMultiInput(data map[string]interface{}, names []string) error
```
- this function allow user to transfer any more parameter the user wanted to.
- this function allow user to choose rules to run.
- this function will execute rules in concurrent model.


### Test Code
- https://github.com/rencalo770/gengine/blob/master/test/pool_test.go
