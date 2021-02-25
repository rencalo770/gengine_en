# inner function
- for handle the golang nil, and make it easy for use, gengine support a inner function isNil()
- when you cannot make sure whether the return value is nil, and it may lead running error, you can use isNil(..) to check

## Attention
- golang cannot handle the pure nil, it means it is not any type

## common rule
- base type will never be nil, it has the own default
- un base type may be nil, test as below：
- array: https://github.com/bilibili/gengine/blob/main/test/map_slice_array/array_nil_value_test.go
- map: https://github.com/bilibili/gengine/blob/main/test/map_slice_array/map_nil_value_test.go
- slice: https://github.com/bilibili/gengine/blob/main/test/map_slice_array/slice_nil_value_test.go



 
   

