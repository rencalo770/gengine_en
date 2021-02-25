### Gengine Introduction
- gengine is a rule engine based on golang and AST(Abstract syntax tree), gengine's grammar is a DSL and very easy!
- gengine open source based on BSD Authorized by BILIBILI(www.bilibili.com) in July 2020. 
- gengine is used in BILIBILI's risk system, content distribution system, AB test and recommend platform.
- gengine is also used by GaoDe(www.amap.com) of alibaba, DiDi(www.didiglobal.com) and many famous companies.
- you can use it in anywhere needs rules or strategies!

gengine has more advantages over Drools which based on java: 

| Compare | drools |  gengine | 
| :--------: | :--------: | :--------------------: |
| execute model | ONLY support sort model | support sort model, concurrent model, mix model and the other model based on them | 
| difficult of writing rules | high,Strongly related to java | low,Custom simple syntax,Weakly related to golang | 
| proformance of execute rules | low, no matter in rule, or between rules, it only support sort model  | high, no matter in rule, or between rules, users can choose different execute model based on needed| 

### why we don't use gopher-lua or js on golang 
- Because we develop our business code by golang, if we use gopher-lua or js on golang, our business logic will escape from golang to lua or javascript, 
and it will make us hava to cost more energy to study lua or javascript, and it also make the business development harder. 
While using gengine, we will use golang to develop all the time, keep the logic in golang, keep the performance as the golang running.
Especially, when you don't want to use gengine any more, it is easy to move your config in gengine to raw golang code

### Thinking in Design
- there is and article of thinking in design about gengine, but it is Chinese: https://xie.infoq.cn/article/40bfff1fbca1867991a1453ac

### How to Use
- go.mod (please use the newest version!) or go vendor.

```
module your_module_name

require github.com/bilibili/gengine v1.5.1
```
the newest tag: https://github.com/bilibili/gengine/tags  


### Github
- https://github.com/bilibili/gengine

### Connection
- issue on github
