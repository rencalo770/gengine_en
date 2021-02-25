## N-M execution mode
- In the NM execution mode, the rules in the rule set are sorted based on priority from high to bottom, and then the rule set is divided into two stages. The first stage is the first N rules with the highest priority, and the second stage is the remaining M Rules, that is, the priority of the rules in the first stage must be higher than the priority of the rules in the second stage, resulting in the following execution sub-modes


### Single instance description and API

- Description

| Execution Mode | Explanation |
| -------- | -------- |
| nSort-mConcurrent | The first n rules are executed sequentially, after the execution is completed, the remaining m rules are executed concurrently |
| nConcurrent-mSort | The first n rules are executed concurrently, after the execution is completed, the remaining m rules are executed in sequence |
| nConcurrent-mConcurrent | The first n rules are executed concurrently, after the execution is completed, the remaining m rules are executed concurrently |
| nSort-mSort | The first n rules are executed in sequence, after the execution is completed, the remaining m rules are executed in sequence; this mode is reduced to a sequential execution mode, which is a normal sequential mode |

- API

| API | Explanation |
| -------- | -------- |
| ``` (g *Gengine) ExecuteNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool) error ```| The first n rules are executed in sequence, after the execution is completed, the remaining m rules are executed concurrently; bool In the control sequence execution stage, when there is a rule execution error, whether to continue execution, true means continue execution |
| ``` (g *Gengine) ExecuteNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool) error``` | The first n rules are executed concurrently, after the execution is completed, the remaining m rules are executed in sequence; bool After controlling the concurrent execution, if there is a rule execution error, whether to continue execution, true means continue execution |
| ``` (g *Gengine) ExecuteNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool) error ``` | The first n rules are executed concurrently, after execution, the remaining m rules are executed concurrently; bool Control the concurrent execution of the first stage, whether to continue executing the second concurrent stage when there is a rule error, true means continue execution|

- Precautions:
1. The parameters n and m must both be greater than 0;
2. The number of rules in the rule set must be greater than or equal to n+m; when the number of rules in the rule set is greater than n+m, the N-M mode is executed for the first n+m rules in the rule set, and the remaining rules are ignored.
3. For detailed test, please see: https://github.com/bilibili/gengine/blob/main/test/n_m_model_test.go

### N-M execution mode based on selection
- We can first select a batch of rules from the rule set based on the specified rule name, and then use the N-M mode to execute, the corresponding API is as follows:

| API | Explanation |
| -------- | -------- |
| ``` (g *Gengine) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error ```| Based on the specified rule name set names, select a batch of rules, The n rules are executed in sequence, and the execution is completed, and the remaining m rules are executed concurrently; bool controls the sequential execution stage, when there is a rule execution error, whether to continue execution, true means continue execution |
| ``` (g *Gengine) ExecuteSelectedNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool, names []string) error``` | Based on the specified rule name set names, select a batch of rules, The n rules are executed concurrently, and the execution is completed, and the remaining m rules are executed in sequence; bool controls whether to continue execution when there is an error in the execution of the rule after the concurrent execution is completed, true means to continue execution |
| ``` (g *Gengine) ExecuteSelectedNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) error ``` | Based on the specified rule name set names, select a batch of rules, before The n rules are executed concurrently, and the execution is completed, and the remaining m rules are executed concurrently; bool controls the concurrent execution of the first stage. When there is a rule error, whether to continue to execute the second concurrent stage, true means to continue execution|

- Precautions:
1. The parameters n and m must both be greater than 0;
2. The number of rules in the selected rule set must be equal to n+m;
3. The rules corresponding to all names in the specified rule name set must exist
4. For detailed test, please see: https://github.com/bilibili/gengine/blob/main/test/n_m_model_test.go


### N-M mode based on gengine pool
- API

| API | Explanation |
| -------- | -------- |
| ``` (gp *GenginePool) ExecuteNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool) (error, map(string)interface()) ```| The first n rules are executed in sequence, and the execution is complete, Concurrently execute the remaining m rules, bool controls the sequential execution stage, whether to continue execution when there is a rule execution error, true means continue execution |
| ``` (gp *GenginePool) ExecuteNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool) (error, map(string)interface())``` | The first n rules are executed concurrently, and the execution is complete, Execute the remaining m rules in sequence, bool controls whether to continue execution when there is an error in the execution of the rule after the concurrent execution is completed, true means to continue execution |
| ``` (gp *GenginePool) ExecuteNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool) (error, map(string)interface()) ``` | The first n rules are executed concurrently, and the execution is complete, The remaining m rules are executed concurrently, and bool controls the concurrent execution of the first stage. When there is a rule error, whether to continue to execute the second concurrent stage, true means continue execution|
| ``` (gp *GenginePool) ExecuteSelectedNSortMConcurrent(nSort, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) (error, map[string]interface()) ```| Based on the specified rule name Set names, select a batch of rules, the first n rules are executed in order, the execution is completed, and the remaining m rules are executed concurrently. The bool controls the sequential execution stage. When there is a rule execution error, whether to continue execution, true means continue execution |
| ``` (gp *GenginePool) ExecuteSelectedNConcurrentMSort(nConcurrent, mSort int, rb *builder.RuleBuilder, b bool, names []string) (error, map[string]interface{))``` | Based on the specified rule name Set names, select a batch of rules, the first n rules are executed concurrently, the execution is completed, the remaining m rules are executed in sequence, bool controls whether to continue execution when there is a rule execution error after the concurrent execution is completed, true means to continue execution |
| ``` (gp *GenginePool) ExecuteSelectedNConcurrentMConcurrent(nConcurrent, mConcurrent int, rb *builder.RuleBuilder, b bool, names []string) (error, map[string]interface()) ``` | Based on the specified rule name Set names, select a batch of rules, the first n rules are executed concurrently, the execution is completed, the remaining m rules are executed concurrently, bool controls the concurrent execution of the first stage, and when there is a rule error, whether to continue to execute the second concurrent stage ,true means continue execution|

Among them, ```map[string]interface()``` is to collect the return value of each strategy (if the strategy has a return value, the key in the map is the name of the strategy, and the value is the specific return value)

 - Precautions
 1. The parameters n and m must both be greater than 0;
 2. The number of rules in the rule set must be greater than or equal to n+m; when the number of rules in the rule set is greater than n+m, the N-M mode is executed for the first n+m rules in the rule set, and the remaining rules are ignored.
 3. The number of rules in the selected rule set must be equal to n+m;
 4. The rules corresponding to all names in the specified rule name set must exist;
 5. For detailed test, please see: https://github.com/bilibili/gengine/blob/main/test/n_m_model_pool_test.go