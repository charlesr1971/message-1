Hi Ben
 
I am trying to get my head around this bit of business logic.
 
I am using FW1 and I have just found out that **controllers** and **services** are _singletons_ [application scope]:
 
https://framework-one.github.io/documentation/4.3/developing-applications/#services-domain-objects-and-persistence
 
Inside one of my controllers I have made a two method calls:
 
```
function list(rc){
    rc.page = 1;
    rc.recordcount = variables.blogService.page(rc.page,0,local.type);
    rc.data = variables.blogService.list();
}
```

The service list function looks like this:

```
function list(){
    return variables.blogs;
}
```
 
So, essentially, the blogService page method uses a DB query to populate:

```
variables.blogs
```
 
And then I immediately return this variable, in the list() method.
 
What worries me, is that if there are two different requests like:
 
**REQUEST 1:**

```
rc.page = 1;
rc.recordcount = variables.blogService.page(rc.page,0,local.type);
rc.data = variables.blogService.list();
```
 
**REQUEST 2:**

```
rc.page = 2;
rc.recordcount = variables.blogService.page(rc.page,0,local.type);
rc.data = variables.blogService.list();
```
 
The list data might get mixed up?
 
My question. Are the two method calls atomic, per request.
 
I know that Coldfusion is multi-threaded, so requests are dealth with, in parallel.
 
I am worried that the following might happen:

```
REQUEST 1 rc.page = 1;
REQUEST 1 rc.recordcount = variables.blogService.page(rc.page,0,local.type);
REQUEST 2 rc.page = 2;
REQUEST 2 rc.recordcount = variables.blogService.page(rc.page,0,local.type);
REQUEST 1 rc.data = variables.blogService.list();
REQUEST 2 rc.data = variables.blogService.list();
```
 
Essentially, the user responsible for REQUEST 1 will receive the REQUEST 2 data.
I can never get my head around this kind of stuff, where components are held in memory. I am pretty sure everything is safe as long as methods are UDFs [functional/stateless methods], but when there is stateful data, in a component held in the application scope, then the alarm bells start ringing. I am not sure why I decided to create a component variables variable, but I think I found it in an FW1 example:

_blogService_

```
component accessors = true{
    variables.blogs = createObject("java","java.util.LinkedHashMap").init();
}
 
function page(page){
    // build variables.blogs struct from DB query
}
 
function list(){
    return variables.blogs;
}
```
 
The two potential solutions I have, are:
 
1. Make services instances, not singletons [remove from the application scope] CANNOT DO THIS AS FW1 THROWS AN ERROR
2. Lock the two method calls, like:

```
cflock (name="listBlogService", type="exclusive", timeout=" 
    rc.recordcount = variables.blogService.page(rc.page,0,local.type);
    rc.data = variables.blogService.list();
}
```

Sorry, for hijacking you on this, but honestly, I cannot think of anyone else, who understands this kind of stuff, better.
