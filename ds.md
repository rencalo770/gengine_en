# Gengine Supported Data Structure

gengine support 4 main data structure , they are golang struct, base-type map、base-type array and base-type slice (golang's feature decide it!), in details:

### struct
**the injected struct must be pointer,or you can't change the field value of struct you inject**, example:

```go
	user := &User{
		Name: "Calo",
		Age:  0,
		Male: true,
	}

	dataContext := context.NewDataContext()
    //inject
	dataContext.Add("User", user)
```
test: https://github.com/rencalo770/gengine/blob/master/test/mutli_rules_test.go 

### map

**gengine only can set map value by key, and injected map must be pointer map, or the map attached to pointer-struct ; when inject non-pointer-map to dataContext, you only can use key to get value, can't set value**
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
测试:https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/map_test.go 

### Array
**gengine only can use index to set value for array or get value from array, and injected array, must be pointer array or the array attached to pointer struct. when inject non-pointer-array to dataContext. you only can get value by index, but not set value**
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
    
    define
   	var AA [2]int
   	AA = [2]int{1, 2}
    
   	dataContext := context.NewDataContext()
   	dataContext.Add("PrintName",fmt.Println)
   	dataContext.Add("AS", AS)
   	//single array inject, must be ptr
   	dataContext.Add("AA", &AA)
```
test: https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/array_test.go

### Slice
**gengine only can use index to set value for slice or get value from array, and injected slice, must be pointer slice or the slice attached to pointer struct. when inject non-pointer-slice to dataContext. you only can get value by index, but not set value**
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
test: https://github.com/rencalo770/gengine/blob/master/test/map_slice_array/slice_test.go 


