### Gengine Introduction
- gengine is a rule engine based on golang and AST(Abstract syntax tree), gengine's grammar is a DSL and very easy!
- gengine open source based on BSD Authorized by BILIBILI(www.bilibili.com) in July 2020. 
- gengine is used in BILIBILI's risk system, content distribution system, AB test and recommend platform.
- you can use it in anywhere needs rules or strategies!

gengine has more advantages over Drools which based on java: 

| Compare | drools |  gengine | 
| :--------: | :--------: | :--------------------: |
| execute model | ONLY support sort model | support sort model, concurrent model, mix model and the other model based on them | 
| difficult of writing rules | high,Strongly related to java | low,Custom simple syntax,Weakly related to golang | 
| proformance of execute rules | low, no matter in rule, or between rules, it only support sort model  | high, no matter in rule, or between rules, users can choose different execute model based on needed| 

### Thinking in Design
- there is and article of thinking in design about gengine, but it is Chinese: https://xie.infoq.cn/article/40bfff1fbca1867991a1453ac


### How to Use
- go.mod (please use the newest version!)

if you not only depend on gengine, but also depend on other framework ,such as grpc, then go.mod should be written like this:

```
module your_module_name

go 1.14

require (
    google.golang.org/grpc v1.22.0
	gengine v1.0.8
)

replace gengine => github.com/rencalo770/gengine v1.1.5
```

if you only depend on gengine, then go.mod is this:

```
module your_module_name

require gengine v1.0.8

replace gengine => github.com/rencalo770/gengine v1.1.5
```



### Github
- https://github.com/rencalo770/gengine


### Version
- gengine's new version is fully compatible with the old version
- gengine's code is continuously updated, we also need your ideas
- https://github.com/rencalo770/gengine/tags

### Connection
- issue on github
