# Execute Model of Rules

As the picture showed,there is 5 class execute model of rule, gengine support the first 3 class model(sort model, concurrent model, mix model), and some other model based on this 3 class model.
![avatar](exe_model.jpg)

### Sort Model
gengine load rules, and execute the rule based on the priorities of the rules, the higher the priority of the rule, the earlier the rule execute. the priorities of rules is the first condition to be considered!

### Concurrent Model
gengine load rules, Concurrently execute the rules, not consider the priorities of rules. this first consider is performance.

### Mix Model
gengine load rules, choose one highest priority rule to execute, and then concurrently execute the last rules. this consider is priority and performance both.

### Explain of Execute Model
- all method of execute rules in this golang file: https://github.com/rencalo770/gengine/blob/master/engine/gengine.go 

| function | execute model | explain | 
| -------- | :--------: | :-------------------- |
|```Execute```|sort model| when b=true, it means when the one rule executed error, it will continue to execute last rules; <br/> when b=false, when the one rule executed error, it will not continue to execute last rules;|
|```ExecuteWithStopTagDirect```|sort model|when b=true, it means when the one rule executed error, it will continue to execute last rules; <br/> when b=false, when the one rule executed error, it will not continue to execute last rules;<br/> when you execute some one rule, and don't want to execute last rules, you can use stopTag to control to stop|
|```ExecuteConcurrent``` |concurrent model|concurrent execute all rules in gengine, if there is error, it will be logged|
|```ExecuteMixModel```|mix model|choose the highest one rule to execute, and then concurrently execute the last rules|
|```ExecuteMixModelWithStopTagDirect```|mix model|choose the highest one rule to execute, and then concurrently execute the last rules<br/>if you execute the one highest priority, and don't want to execute last rules, you can use stopTag to control to stop|
|```ExecuteSelectedRules```|sort model| user can select rules to execute by specifying the rules' name, and the rules will be executed based on priority|
|```ExecuteSelectedRulesConcurrent```|mix model| user can select rules to execute by specifying the rules' name, and rules will be executed concurrently|

```ExecuteSelectedRules``` and ```ExecuteSelectedRulesConcurrent``` will be explain later.






