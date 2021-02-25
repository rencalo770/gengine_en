# Advanced extension section

### Fully hot reloading implementation-background
- Up to now, we can see that the APIs we use in gengine are all pre-defined before gengine is started, and then when the initialization is complete, it can be injected into gengine to be used. This mode of use is of course good, and It can basically meet the needs of all business scenarios: For example, users can define several interfaces in advance, and then use these defined interfaces to obtain different data required by the rule during the execution process (based on the name Way to access data indicators).
- But it is possible that all the functions currently defined cannot meet the requirements, and the user must define new methods or functions to achieve new functions.

### Reality
Golang is a statically typed language, not as flexible as java. If you want to dynamically load code and execute it, you can only use plug-ins. gengine wraps the call implementation of plugin, and uses plugin to load and execute code hot.

### Description
- The method of dynamically loading code packaged by gengine only needs to comply with a few regulations, and it will be extremely simple to use
- gengine has supported a fully dynamic loading implementation based on golang plugin since v1.4.9

### golang plugin use
- Plug-in writing

```go
//package must be main
package main


//In order to demonstrate the complete use of go plugin that has nothing to do with gengine, an interface is needed
type Man interface {
SaveLive() error
}

type SuperMan struct {

}

func (g *SuperMan) SaveLive() error {

println("execute finished...")
return nil
}

//go build -buildmode=plugin -o=plugin_M_m.so plugin_superman.go

// exported as symbol named "M", must start with uppercase
var M = SuperMan{}
```

- Notes for plug-in writing
1. The package name must be main
2. The exported variable must start with uppercase and cannot have the same name with other struct or interface definitions
3. Execute the ```go build -buildmode=plugin -o=plugin_M_m.so plugin_superman.go'' command to generate the .so plugin file


- Plug-in usage test

```go
func Test_pligin(t *testing.T) {

dir, err := os.Getwd()
if err!=nil {
panic(err)
}

// load module plug-in, you can also use go http.Request to download from remote to local, and perform different functions dynamically while loading
// 1. open the so file to load the symbols
plug, err := plugin.Open(dir + "/plugin_M_m.so")
if err != nil {
panic(err)
}
println("plugin opened")

// 2. look up a symbol (an exported function or variable)
// in this case, variable Greeter
m, err := plug.Lookup("M") //uppercase
if err != nil {
panic(err)
}

// 3. Assert that loaded symbol is of a desired type
man, ok := m.(Man)
if !ok {
fmt.Println("unexpected type from module symbol")
os.Exit(1)
}

// 4. use the module
if err := man.SaveLive(); err != nil {
println("use plugin man failed, ", err)
}
}

```

### gengine is based on plugin's hot loading implementation
1. It can be seen from the use of the above plug-ins that the use of plug-ins requires several steps: a. Define the plug-in go file, b. Command to generate the .so file, c. Open load the plug-in d. Lookup to find the api exported in the plug-in definition, e. Interface type conversion f. Use the functions provided by the plug-in
2. To use the plugin in gengine, you need steps a and b, and then tell the location of the .so file of the gengine plugin, you can use it in the gengine rule configuration
3. However, to be able to use the plugin correctly in gengine, you need to comply with a few specifications (requirements): a. The generated .so file must be in the form of plugin_exportName_apiName.so, where exportName is the export name in the plugin implementation. Start with uppercase, apiName is the plugin name used in the gengine rule configuration
4. The hot loading interfaces provided in gengine are all thread-safe
5. Please see the test below

- Single instance plugin loading test
```go
func Test_plugin_with_gengine(t *testing.T) {

dir, err := os.Getwd()
if err!=nil {
panic(err)
}

dc := context.NewDataContext()
//3.load plugin into apiName, exportApi
_, _, e := dc.PluginLoader( dir + "/plugin_M_m.so")
if e != nil {
panic(e)
}

dc.Add("println", fmt.Println)
ruleBuilder := builder.NewRuleBuilder(dc)
err = ruleBuilder.BuildRuleFromString(`
rule "1"
begin
To
//this method is defined in plugin
err = m.SaveLive()

if isNil(err) {
println("err is nil")
}
end
`)

if err != nil {
panic(err)
}
gengine := engine.NewGengine()
err = gengine.Execute(ruleBuilder, false)

if err!=nil {
panic(err)
}
}
```

- pool plugin test

```go
func Test_plugin_with_pool(t *testing.T) {

rule :=`
rule "1"
begin
To
//this method is defined in plugin
err = m.SaveLive()

if isNil(err) {
println("err is nil")
}
end`

apis := make(map[string]interface{})
apis["println"] = fmt.Println
pool, e := engine.NewGenginePool(1, 2, 1, rule, apis)
if e != nil {
panic(e)
}

dir, err := os.Getwd()
if err!=nil {
panic(err)
}

e = pool.PluginLoader( dir + "/plugin_M_m.so")
if e != nil {
panic(e)
}
data := make(map[string]interface{})
e, _ = pool.Execute(data, true)
if e != nil {
panic(e)
}

//twice execute
e, _ = pool.Execute(data, true)
if e != nil {
panic(e)
}
}

```

### Test location
- Test code location https://github.com/bilibili/gengine/tree/main/test/plugin