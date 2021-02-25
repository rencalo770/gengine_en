# Execute Model of Rules

As the picture showed,there is 5 class execute model of rule, gengine support the first 3 class model(sort model, concurrent model, mix model), and some other model based on this 3 class model.
![avatar](exe_model.jpg)

### Sort Model
gengine load rules, and execute the rule based on the priorities of the rules, the higher the priority of the rule, the earlier the rule execute. the priorities of rules is the first condition to be considered!

### Concurrent Model
gengine load rules, Concurrently execute the rules, not consider the priorities of rules. this first consider is performance.

### Mix Model
gengine load rules, choose one highest priority rule to execute, and then concurrently execute the last rules. this consider is priority and performance both.

### Inverse Mix Model
gengine load rules,  choose one highest priority  N-1 rule to execute, and then execute the last rules. 

### Explain of Execute Model
- all method of execute rules in this golang file: https://github.com/bilibili/gengine/blob/main/engine/gengine.go and https://github.com/bilibili/gengine/blob/main/engine/gengine_pool.go







