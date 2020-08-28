# Advanced extension

### Background
- until now, all API we used in gengine is that is Predefined before we start gengine,then init and inject into gengine.this model is good and it also can meet our work needed: user can pre define some API, and get different data by the same API with different parameters(based on name to get data or service).
- but in this case: predefined API can't meet the user needs.

### Reality
if you want use golang to load code dynamically,you have to use plugin in golang. gengine wrapped the golang's plugin module to implement loading code dynamically. 

### Other Description
- gengine's dynamic loading code function will be easy to use.
- The specific instructions for use will be updated after the tag version is pushed.
- gengine will implement the dynamic loading code function  in tag version v1.2.1 or v1.2.2






