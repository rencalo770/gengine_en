# Rule execution method based on user selection
### Background
We have observed this requirement during the use of gengine:<br/>
-Assuming that there are 10 rules in gengine, the rule name number is from "1" to "10", when a request comes, the user clearly knows which rules he needs to execute.
-**gengine supports users to use the rule name to select the rules to be executed**, which not only meets the needs of users, but also has higher performance.
-Each execution mode in gengine has a corresponding selection mode

### Specific method

#### ExecuteSelectedRules method
```go
func (g *Gengine)ExecuteSelectedRules(rb * builder.RuleBuilder, names []string)
```
-The names parameter is an array of multiple rule names. For example, when the user passes in ```name :=[]string{"1", "7", "3"}```, gengine will The 3 rules are executed based on the priority order of the 3 rules selected by the user.
-If the rule name passed in by the user does not exist, gengine will log it during execution.

#### ExecuteSelectedRulesConcurrent method
```go
func (g *Gengine)ExecuteSelectedRulesConcurrent(rb * builder.RuleBuilder, names []string)
```
-The names parameter is an array of multiple rule names. For example, when the user passes in ```name :=[]string{"1", "7", "3"}```, gengine does not The priority of the rules will be considered, and these 3 rules will be executed concurrently.
-If the rule name passed in by the user does not exist, gengine will log it during execution.

#### ExecuteSelectedRulesMixModel method
```go
func (g *Gengine) ExecuteSelectedRulesMixModel(rb *builder.RuleBuilder, names []string) error
```
-The names parameter, which is an array of multiple rule names. For example, when the user passes in ```name :=[]string{"1", "7", "3"}```, gengine is mixed The pattern enforces these 3 rules.
-If the rule name passed in by the user does not exist, gengine will log it during execution.

#### ExecuteSelectedInverseMixModel method
```go
func (g *Gengine) ExecuteSelectedInverseMixModel(rb *builder.RuleBuilder, names []string) error {
```
-The names parameter is an array of multiple rule names. For example, when the user passes in ```name :=[]string{"1", "7", "3"}```, gengine reverses The mixed mode executes these 3 rules.
-If the rule name passed in by the user does not exist, gengine will log it during execution.

#### Selection mode based on N-M mode
```
func (g *Gengine) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error
func (g *Gengine) ExecuteSelectedNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool, names []string) error
func (g *Gengine) ExecuteSelectedNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error
```

### Implementation
-The execution method name with the word select is the selection mode https://github.com/bilibili/gengine/blob/main/engine/gengine.go

### Test case
-https://github.com/bilibili/gengine/blob/main/test/exceute_selected_rules_test.go