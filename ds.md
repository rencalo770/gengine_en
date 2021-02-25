# Data structure supported by gengine

gengine mainly supports four data structures, namely golang's struct, and basic types of map, array and slice. In order to maintain simplicity and language independence, the following operations of these four structures are supported:

### Inject struct
**The injected struct must be injected in the form of a pointer, otherwise the attribute value of the structure cannot be changed in the rules**, an example of the injected structure is as follows:

```go
user := &User{
Name: "Calo",
Age: 0,
Male: true,
}

dataContext := context.NewDataContext()
//inject
dataContext.Add("User", user)
```
Test: https://github.com/bilibili/gengine/blob/main/test/mutli_rules_test.go

### Inject map

gengine can only take or set values ​​for map based on key

```go
//define
type MS struct {
    MII *map[int]int
    MSI map[string]int
    MIS map[int]string
}
    
//init
MS := &MS{
MII: &map[int]int{1: 1},
MSI: map[string]int{"hello": 1},
MIS: map[int]string{1: "helwo"},
}
    
//define
var MM map[int]int
MM = map[int]int{1:1000,2:1000}
    
//inject
dataContext := context.NewDataContext()
dataContext.Add("MS", MS)
//single map inject, must be ptr
dataContext.Add("MM", &MM)
```
Test: https://github.com/bilibili/gengine/blob/main/test/map_slice_array/array_test.go

### Inject array
**gengine can only get or set the value of an array based on index, and the injected array must be a pointer array, or an array attached to a structure pointer. When injecting dataContext into a non-pointer array, you can only use index to get Value instead of setting value**
```go
    
//define
type AS struct {
     MI *[3]int
     MM [4]int
}
    
//init
AS := &AS{
   MI: &[3]int{},
   MM: [4]int{},
  }
    
//define
var AA [2]int
AA = [2]int{1, 2}
    
dataContext := context.NewDataContext()
dataContext.Add("PrintName",fmt.Println)
dataContext.Add("AS", AS)
//single array inject, must be ptr
dataContext.Add("AA", &AA)
```
Test: https://github.com/bilibili/gengine/blob/main/test/map_slice_array/array_test.go

### Inject slice

gengine can only take or set values ​​for slices based on index

```go
    
//define
type SS struct {
MI []int
MM *[]int
    }

    //init
SS := &SS{
MI: []int{1,2,3,4},
MM: &[]int{9,1,34,5},
}
    
    //define
var S []int
S = []int{1, 2, 3}

dataContext := context.NewDataContext()
dataContext.Add("PrintName",fmt.Println)
dataContext.Add("SS", SS)
   //single slice inject, must be ptr
dataContext.Add("S", &S)

```
Test: https://github.com/bilibili/gengine/blob/main/test/map_slice_array/slice_test.go