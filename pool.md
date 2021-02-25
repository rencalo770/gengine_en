# Rule engine pool
"Rule engine pool" refers to a pool with multiple gengine instances. You can compare the "rule engine pool" to the "database connection pool", all for solving performance and concurrency security issues.

### Background

- A series of rules loaded into a gengine instance are stateful when executed.
- When a gengine instance is used as a module called in the service core, when there are few rules, the time for a request to execute all the rules is very short, and the time for the state of the rules executed based on the current request is also very short, so it is not Will cause any problems;
- But when dozens or even hundreds of rules are loaded in a gengine instance, the time for a request to execute all the rules will become longer, especially in the case of high QPS, and the current request has not been executed. When returning, the next request has already arrived. At this time, the next request is very likely to destroy the state of the current request execution rule and eventually lead to unpredictable results.
- In order to solve this problem, the gengine framework imitates the "database connection pool" and implements a "gengine pool".
- All APIs of pool are thread-safe

### Code location
https://github.com/bilibili/gengine/blob/main/engine/gengine_pool.go

### Engine pool initialization
```go
//Initialize a rule engine pool
func NewGenginePool(poolMinLen ,poolMaxLen int64, em int, rulesStr string, apiOuter map[string]interface{})
```
#### Parameter Description
- poolMinLen The number of gengine instances initially instantiated in the pool
- poolMaxLen The maximum number of gengine instances that can be instantiated in the pool, and poolMaxLen> poolMinLen; when poolMinLen instances are not enough, up to (poolMaxLen-poolMinLen) gengine instances can be instantiated
- em rule execution mode, em can only be equal to 1, 2, 3, 4; when em=1, it means that the rules are executed in sequence, when em=2, it means that the rules are executed concurrently, and when em=3, it means that the rules are mixed mode execution. When em=4, it means the inverse mixed mode execution rule; when using ```ExecuteSelectedRules``` and ```ExecuteSelectedRulesConcurrent`'' to specify the specific execution mode method execution rules, this parameter is automatically invalid
-rulesStr to initialize all rule strings
-apiOuter needs to be injected into all apis used in gengine, it is best to inject only some public stateless functions or parameters; for those parameters specifically related to a certain request (execution), use ```data when executing the rule method map[string]interface{} ``` injection; this will help state management.

#### Method of executing rules in pool
- The pool provides some flag fields em based on the execution mode to control the specific mode of some general methods. Although it is convenient for some scenarios that require frequent mode switching, it is also used for scenarios that only require a certain mode. There are doubts. Therefore, gengine pool implements the following strategies
- ***The method in the single instance of gengine, the corresponding pool mode will also have a method of the same name with exactly the same function. In this way, it is easy for users to test (in a single instance) and easy for users to use ( The method with the same name in the pool), when the method with the corresponding name, the em field when initializing the pool is invalid***
- The specific correspondence is as follows:

| Methods in ```gengine.go``` (top) <br/> Corresponding methods in ```gengine_pool.go``` (bottom) | Method meaning |
| -------- | -------- |
|```func (g *Gengine) Execute(rb *builder.RuleBuilder, b bool) error```<br/> ****``` func (gp *GenginePool) Execute(data map[string]interface (), b bool) (error, map[string]interface()) ```**** | Sequential execution mode, <br/>b=true means that even if a rule goes wrong, continue to execute the remaining rules ;<br/>If b=false means that if a rule fails, then the following rules will not be executed |
|```func (g *Gengine) ExecuteWithStopTagDirect(rb *builder.RuleBuilder, b bool, sTag *Stag) error ``` <br/> ****```func (gp *GenginePool)ExecuteWithStopTagDirect(data map [string]interface{}, b bool, sTag *Stag) (error, map[string]interface{}) ```****|Execute all rules in order,<br/>b=true means even a certain rule If an error occurs, the remaining rules will continue to be executed; <br/>If b=false means that if a rule fails, the subsequent rules will not be executed; <br/>If sTag=true, the subsequent rules will not be executed. If sTag=false, continue to execute the following rules|
|```func (g *Gengine) ExecuteConcurrent(rb *builder.RuleBuilder) error ``` <br/> ****``` func (gp *GenginePool)ExecuteConcurrent(data map[string]interface()) (error, map[string]interface()) ```****|Concurrent mode, execute all rules concurrently|
|```func (g *Gengine) ExecuteMixModel(rb *builder.RuleBuilder) error ``` <br/> ****```func (gp *GenginePool)ExecuteMixModel(data map[string]interface()) (error, map[string]interface()) ```****|Mixed mode, first execute a rule with the highest priority, and then execute all remaining rules concurrently|
|```func (g *Gengine) ExecuteMixModelWithStopTagDirect(rb *builder.RuleBuilder, sTag *Stag) error ``` <br/> ****```func (gp *GenginePool) ExecuteMixModelWithStopTagDirect(data map[string] interface(), sTag *Stag) (error, map[string]interface()) ```****|Mixed mode, <br/>first execute a rule with the highest priority, and then execute all the rest concurrently Rules;<br/>If sTag is set to true in the first rule to be executed, the remaining rules will not be executed|
|```func (g *Gengine) ExecuteSelectedRules(rb *builder.RuleBuilder, names []string) error ``` <br/> ****``` func (gp *GenginePool) ExecuteSelectedRules(data map[string ]interface{}, names []string) (error, map[string]interface{}) ```****|Select a batch of rules based on the rule name, and execute them in sequential mode.<br/>Even if there are If the rule execution fails, all remaining rules will continue to be executed|
|```func (g *Gengine) ExecuteSelectedRulesWithControl(rb *builder.RuleBuilder, b bool, names []string) error ``` <br/> ****```func (gp *GenginePool) ExecuteSelectedRulesWithControl(data map[string]interface{}, b bool, names []string) (error, map[string]interface{}) ```****|Select a batch of rules based on the rule name and execute them in sequential mode,<br/ >Even if there is a rule execution error during the execution, all remaining rules will continue to be executed;<br/>b=true means that even if a rule goes wrong, the remaining rules will continue to be executed;<br/>if b=false Indicates that if a rule fails, the following rules will not be executed|
|```func (g *Gengine) ExecuteSelectedRulesWithControlAndStopTag(rb *builder.RuleBuilder, b bool, sTag *Stag, names []string) error ``` <br/> ****```func (gp *GenginePool )ExecuteSelectedRulesWithControlAndStopTag(data map[string]interface{}, b bool, sTag *Stag, names []string) (error, map[string]interface{}) ```****|Select a batch of rules based on the rule name ,Sequential mode execution,<br/>even if there is a rule execution error during the execution process, all the remaining rules will continue to be executed; <br/>b=true means that even if a rule fails, the remaining rules will continue to be executed; <br/>If b=false means that if a rule fails, the following rules will not be executed; <br/>If sTag is set to true in the first executed rule, the remaining rules will not be executed |
|```func (g *Gengine) ExecuteSelectedRulesConcurrent(rb *builder.RuleBuilder, names []string) error ``` <br/> ****```func (gp *GenginePool) ExecuteSelectedRulesConcurrent(data map[string ]interface{}, names []string) (error, map[string]interface{}) ```****|Select a batch of rules based on the rule name, and execute the selected rules concurrently|
|```func (g *Gengine) ExecuteSelectedRulesMixModel(rb *builder.RuleBuilder, names []string) error ``` <br/> ****```func (gp *GenginePool) ExecuteSelectedRulesMixModel(data map[string ]interface{}, names []string) (error, map[string]interface{}) ```****|Select a batch of rules based on the rule name, and execute the selected rules in mixed mode|
|```func (g *Gengine) ExecuteInverseMixModel(rb *builder.RuleBuilder) error ``` <br/> ****``` func (gp *GenginePool) ExecuteInverseMixModel(data map[string]interface()) (error, map[string]interface{}) ```**** |Inverse mixed execution mode, execute all rules|
|```func (g *Gengine) ExecuteSelectedRulesInverseMixModel(rb *builder.RuleBuilder, names []string) error ``` <br/> ****``` func (gp *GenginePool)ExecuteSelectedRulesInverseMixModel(data map[string]interface{}, names []string) (error, map[string ]interface{}) ```****|Select a batch of rules based on the rule name, reverse the mixed execution mode, and execute the selected rules|
|```func (g *Gengine) ExecuteNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool) error ``` <br/> ****```func (gp *GenginePool) ExecuteNSortMConcurrent(nSort , mConcurrent int, b bool, data map[string]interface{}) (error, map[string]interface{}) ```****|NM mode, the first N rules are executed in sequence, the next M Rules are executed concurrently, and N+M is greater than or equal to the length of the rule set|
|```func (g *Gengine) ExecuteNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool) error ``` <br/> ****```func (gp *GenginePool) ExecuteNConcurrentMSort(nSort , mConcurrent int, b bool, data map[string]interface{}) (error, map[string]interface{}) ```****|NM mode, the first N rules are executed concurrently, the next M The rules are executed in order, and N+M is greater than or equal to the length of the rule set|
|```func (g *Gengine) ExecuteNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool) error ```<br/> ****```func (gp *GenginePool) ExecuteNConcurrentMConcurrent(nSort , mConcurrent int, b bool, data map[string]interface{}) (error, map[string]interface{}) ```****|NM mode, the first N rules are executed concurrently, the next M Rules are also executed concurrently, and N+M is greater than or equal to the length of the rule set|
|```func (g *Gengine) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error ```<br/> ****```func (gp * GenginePool) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, b bool, names []string, data map[string]interface{}) (error, map[string]interface{}) ```****|Select one based on the rule name Batch rules, NM mode, the first N rules are executed sequentially, the next M rules are executed concurrently, and N+M is greater than or equal to the length of the selected rule set |
|```func (g *Gengine) ExecuteSelectedNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool, names []string) error ``` <br/> ****```func (gp * GenginePool) ExecuteSelectedNConcurrentMSort(nSort, mConcurrent int, b bool, names []string, data map[string]interface{}) (error, map[string]interface{}) ```**** |Select one based on the rule name Batch rules, NM mode, the first N rules are executed concurrently, the next M rules are executed sequentially, and N+M is greater than or equal to the length of the selected rule set|
|```func (g *Gengine) ExecuteSelectedNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error ```<br/>****```func (gp * GenginePool) ExecuteSelectedNConcurrentMConcurrent(nSort, mConcurrent int, b bool, names []string, data map[string]interface{}) (error, map[string]interface{}) ```****|Select one based on the rule name Batch rules, NM mode, the first N rules are executed concurrently, the next M rules are also executed concurrently, and N+M is greater than or equal to the length of the selected rule set|
                                                                          
- The second return value of the pool execution method is the return value of the return statement after each rule in the rule set is executed
                                                                          
### How to set poolMaxLen size
- How to set the maximum number of instances in the engine pool
- poolMinLen = poolMaxLen / 2
- poolMaxLen = cpu_core * 5 to cpu_core * 10
                                                                          
### Dynamic real-time update rules
```go
     //Update the rules in the gengine instance in the rule engine pool
     func (gp *GenginePool)UpdatePooledRules(ruleStr string) error
```
This method allows users to update the rules in real time when the "rule engine pool" provides services externally, and is thread-safe
                                                                          
### Clear rules
```go
//Clear all rules
func (gp *GenginePool)ClearPoolRules()
```
This method allows users to clear rules in real time when the "rule engine pool" provides external services, and is thread-safe
                                                                          
### Set execution mode
```go
//Set the execution mode
func (gp *GenginePool)SetExecModel(execModel int) error
```
                                                                          
- This method allows users to change the rule execution mode in real time when the "rule engine pool" provides external services, and is thread-safe
- There are only three effective execution functions for execModel (may be added in the future, but the method name must end with WithSpecifiedEM), execModel can be 1, 2, 3, 4
1. ```func (gp *GenginePool) ExecuteRulesWithSpecifiedEM(reqName string, req interface(), respName string, resp interface()) (error, map(string)interface()) ```, this method only allows req injection And resp parameters
2. ```func (gp *GenginePool) ExecuteRulesWithMultiInputWithSpecifiedEM(data map(string)interface()) (error, map(string)interface()) ```, this method allows to inject any number of parameters before running
3. ```func (gp *GenginePool) ExecuteSelectedWithSpecifiedEM(data map[string]interface(), names []string) (error, map[string]interface()) ```, this method is allowed in selection mode Inject any number of parameters before running
                                                                          
### Other methods that may be important
                                                                          
#### Check whether the rule exists
``` func (gp *GenginePool) IsExist(ruleName string) bool ``` Check whether a rule exists in the pool
                                                                          
#### Get the priority of the rule
``` func (gp *GenginePool) GetRuleSalience(ruleName string) (int64, error) ``` If error!=nil means there is no such rule; otherwise, you can get the priority of the specified rule
                                                                          
#### Get rule description information
``` func (gp *GenginePool) GetRuleDesc(ruleName string) (string, error) ``` If error!=nil means there is no such rule; otherwise, you can get the description of the specified rule
                                                                          
#### Get the number of different rules in the pool
``` func (gp *GenginePool) GetRulesNumber() int ```