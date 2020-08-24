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
#### 参数说明
- poolMinLen, the min number of gengine instance in pool 
- poolMaxLen, the min number of gengine instance in pool, and poolMaxLen > poolMinLen; when there is not enough gengine instance to execute, you can init additional poolMaxLen-poolMinLen gengine instance.
- em 规则执行模式,em只能等于1、2、3; em=1时,表示顺序执行规则,em=2的时候,表示并发执行规则,em=3的时候,表示混合模式执行规则;当使用```ExecuteSelectedRulesWithMultiInput```和```ExecuteSelectedRulesConcurrentWithMultiInput```方法执行规则时,此参数自动失效
- rulesStr要初始化的所有的规则字符串
- apiOuter需要注入到gengine中使用的所有api

### 动态化更新规则
```go 
//更新规则引擎池中gengine实例中的规则
func (gp *GenginePool)UpdatePooledRules(ruleStr string) error
```
此方法允许用户在"规则引擎池"对外提供服务的时候,实时的更新规则,且线程安全

#### 参数说明
- ruleStr 新的用户需要加载的规则的字符串

### 清空规则
```go
//清空所有规则
func (gp *GenginePool)ClearPoolRules()
```
此方法允许用户在"规则引擎池"对外提供服务的时候,实时的清空规则,且线程安全

### 设置执行模式
```go
//设置执行模式
func (gp *GenginePool)SetExecModel(execModel int) error 
```
- 此方法允许用户在"规则引擎池"对外提供服务的时候,实时的改变规则执行模式,且线程安全

#### 参数说明
- execModel 执行模式编号,只能为1、2或3中的一个值
- 当使用```ExecuteSelectedRulesWithMultiInput```和```ExecuteSelectedRulesConcurrentWithMultiInput```方法执行规则时,此参数自动失效
### 执行规则

#### ExecuteRules方法
 ```go
func (gp *GenginePool)ExecuteRules(reqName string, req interface{}, respName string, resp interface{}) error
```
- reqName 对具体值req在gengine中使用的时候的命名
- req 具体的用户请求数据
- respName 对具体值resp在gengine中使用的时候的命名
- resp 具体的用户返回数据
- 此方法的局限是,仅能传入两个临时参数

#### ExecuteRulesWithMultiInput方法
```go
func  (gp *GenginePool)ExecuteRulesWithMultiInput(data map[string]interface{}) error
```
- 此方法是为了解决```ExecuteRules```方法的局限的一个新方法,其他的内部实现和其完全一样.
- 用户可以传入任意多个临时参数到gengine中,但传入的参数要先放到map中

#### ExecuteRulesWithStopTag方法
```go
func (gp *GenginePool)ExecuteRulesWithStopTag(reqName string, req interface{}, respName string, resp interface{}, stag *Stag) error
```
- 此方法和```ExecuteRules```唯一不同的地方在于,在顺序执行模式和混合模式下,用户可以使用stag来控制是否继续执行后续规则

#### ExecuteRulesWithMultiInputAndStopTag方法
```go
func (gp *GenginePool)ExecuteRulesWithMultiInputAndStopTag(data map[string]interface{}, stag *Stag) error 
```
- 此方法继承了```ExecuteRulesWithMultiInput``` 可以传入任意多个临时参数的优点
- 同时继承了```ExecuteRulesWithStopTag```可以在顺序执行模式和混合模式下，用户可以使用stag来控制是否继续执行后续规则的优点
- v1.1.8中引入

#### ExecuteSelectedRulesWithMultiInput方法
```go
func  (gp *GenginePool)ExecuteSelectedRulesWithMultiInput(data map[string]interface{}, names []string) error
```
- 此方法允许用户输入任意多个临时参数
- 此方法允许用户在池中选择指定的规则去执行
- 此方法顺序执行用户选中的规则

#### ExecuteSelectedRulesConcurrentWithMultiInput方法
```go
func  (gp *GenginePool)ExecuteSelectedRulesConcurrentWithMultiInput(data map[string]interface{}, names []string) error
```
- 此方法允许用户输入任意多个临时参数
- 此方法允许用户在池中选择指定的规则去执行
- 此方法并发执行用户选中的规则

### 测试代码
- https://github.com/rencalo770/gengine/blob/master/test/pool_test.go



















