#关于Self=This,和function一些问题的探讨
function在Javascript里是一个很有趣的东西,它是对象,  
但对象本身是不可以被调用的,对象有的是方法(method)也就是function.  
但这个'对象'非但可以被调用,而且可以有自己属性!
例如:

```js
var f1=function(){return 'f1'};
var f2=f1;
f2.att='att'
console.log(f1.att);//'att'
console.log(f2.att);//'att'
f2=function(){return 'f2'};
f2();//f2
f1();//f1
typeof f1;//'function'
typeof f1.att//'string'
```

其次,function的上下文很奇怪,  
function里面的this,是对其上下文的引用.  
**如果这个function是直接声明** (无论局部还是全局,var还是function Name(){})  
包括任何地方的匿名function, **this就会指向全局对象**, 在浏览器端就是window

只有 **当function作为对象的方法时,this才会指向直属对象**,  
另外一种情况是作为构造函数时, **利用运算符new创造新对象,**  
**这时this是指向这个新对象!**

##第一种情况,当i是一个用对象字面量声明的对象
self主要用于对象内方法的函数引用对象本身,当然这样的情况很少,因为本身这个对象不可以复制,除非用特殊手段,引用本身的话直接写名字就好了

```js
var i = {
    att: "i's att",
    con: function() {
        var consvar = "i.con's var";
        var self = this;
        console.log('~~~i.con的this:')
        console.log(this);
        return (function() {
            console.log("~~~i.con返回一个匿名函数的this");
            console.log(this);
            console.log("~~~这个匿名函数依然能访问i.con的作用域的局部变量consvar")
            console.log(consvar);
            console.log("~~~在i.con里声明self=this,就可以在返回的匿名函数里通过self找回i.con的this")
            console.log(self);
            console.log('~~~用this访问att属性是不行的,因为匿名函数上下文在全局');
            console.log(this.att);
            console.log('~~~指定i.att当然ok');
            console.log(i.att);
            console.log('~~~或者,利用self.att也可以');
            console.log(self.att);
            return '~~~done';
        })();
    }
};
```

```js
i.con()
```
>  
> i.con的this:  
> > **Object {att: "i's att", con: function}**  
>   
> i.con返回一个匿名函数的this  
> > **Window**  
> 
> 这个匿名函数依然能访问i.con的作用域的局部变量consvar  
> > **i.con's var**  
>   
> 在i.con里声明self=this,就可以在返回的匿名函数里通过self找回i.con的this  
> > **Object {att: "i's att", con: function}**  
>   
> 用this访问att属性是不行的,因为匿名函数上下文在全局  
> > **undefined**  
>   
> 指定i.att当然ok  
> > **i's att**  
>   
> 或者,利用self.att也可以  
> > **i's att**  
> **done**  


##第二种情况,就是通过一个构造函数来创建对象i
通过构造函数, **可以解决第三种情况中对象里的对象不能访问上一层的属性,**  
我们只需要利用好self=this和原始构造函数的作用域就可以另不同层级对象间互相访问了

```js
var iConstructor = function(e) {
    var iCatt = this.att = e;
    var self = this;
    console.log(this);
    this.iObjConstructor = function(e) {
        console.log(this);
        this.att = "from outside " + e;
        this.ihiSelf = self;
        this.ihiThis = this;
        return;
    };
    this.iObjConstructor.prototype.con = function() {
        console.log('~~~这是原始构造函数里的另一个构造函数原型的this,指向后者创造的对象')
        console.log(this);
        console.log('~~~这是原始构造函数里的另一个构造函数原型访问原始构造函数声明的self,指向前者创造的对象');
        console.log(self);
        console.log('~~~原始构造函数里的另一个构造函数原型可以访问原始构造函数声明的局部变量,这个并不能通过外部修改');
        console.log('iCatt is ' + iCatt);
        console.log('~~~这个是原始构造函数创造的对象的属性,是可以修改的')
        console.log('self.att is ' + self.att);

        function iObjProtoFun() {
            console.log('~~~构造函数里的另一个构造函数原型里一个函数的this');
            console.log(this);
            console.log('~~~这个函数可以访问原始构造函数的self.att,self.att是原始构造函数创造的对象的属性');
            console.log(self.att);
        };
        iObjProtoFun();
        return 'i.iObj.con done';
    };
    this.iObj = new this.iObjConstructor(e);
    return;
};
iConstructor.prototype.con = function() {
    var consvar = "iConstructor.prototype.con var";
    var self = this;
    console.log('~~~i.con的this:')
    console.log(this);
    return (function() {
        console.log("~~~i的原型方法con返回一个匿名函数的this");
        console.log(this);
        console.log("~~~这个匿名函数能访问这个原型方法con作用域的局部变量consvar")
        console.log(consvar);
        console.log("~~~利用self=this可以在匿名函数作用域找回构造函数创造的对象")
        console.log(self);
        console.log('~~~用this访问att属性是不行的,因为匿名函数上下文在全局');
        console.log(this.att);
        console.log('~~~指定i.att当然ok');
        console.log(i.att);
        console.log('~~~或者,利用self.att也可以');
        console.log(self.att);
        console.log('~~~最后,尝试在构造函数的原型访问构造函数作用域里的局部变量iCatt,是不可以的');
        try {
            console.log(iCatt);
        } catch (err) {
            return err;
        };
    })();
};
```

```js
var i = new iConstructor("i.att");
i.con()
```
> i.con的this:  
> > **iConstructor {att: "i.att", iObjConstructor: function, iObj: iObjConstructor, con: function}**  
>  
> i的原型方法con返回一个匿名函数的this  
> > **Window**  
>  
> 这个匿名函数能访问这个原型方法con作用域的局部变量consvar  
> > **iConstructor.prototype.con var**  
>  
> 利用self=this可以在匿名函数作用域找回构造函数创造的对象  
> > **iConstructor {att: "i.att", iObjConstructor: function, iObj: iObjConstructor, con: function}**  
>  
> 用this访问att属性是不行的,因为匿名函数上下文在全局  
> > **undefined**  
>  
> 指定i.att当然ok  
> > **i.att**  
>  
> 或者,利用self.att也可以  
> > **i.att**  
>  
> 最后,尝试在构造函数的原型访问构造函数作用域里的局部变量iCatt,是不可以的  
> > **ReferenceError {}**  
>  
```js  
i.iObj.con()
```
> 
> 这是原始构造函数里的另一个构造函数原型的this,指向后者创造的对象  
> > **iObjConstructor {att: "from outside i.att", ihiSelf: iConstructor, ihiThis: iObjConstructor, con: function}**  
>  
> 这是原始构造函数里的另一个构造函数原型访问原始构造函数声明的self,指向前者创造的对象  
> > **iConstructor {att: "i.att", iObjConstructor: function, iObj: iObjConstructor, con: function}**  
>  
> 原始构造函数里的另一个构造函数原型可以访问原始构造函数声明的局部变量,这个并不能通过外部修改  
> > **iCatt is i.att**  
>             
> 这个是原始构造函数创造的对象的属性,是可以修改的  
> > **self.att is i.att**  
>                
> 构造函数里的另一个构造函数原型里一个函数的this  
> > **Window**  
>              
> 这个函数可以访问原始构造函数的self.att,self.att是原始构造函数创造的对象的属性  
> > **i.att**  
> **i.iObj.con done**  
>  
```js
i.iObj.ihiSelf
```
> **iConstructor {att: "i.att", iObjConstructor: function, iObj: iObjConstructor, con: function}**  
```js
i.iObj.ihiThis
```
> **iObjConstructor {att: "from outside i.att", ihiSelf: iConstructor, ihiThis: iObjConstructor, con: function}**  
>  

##第三种情况,i是一个普通声明的function
通过在内部建立一个局部对象iObj观察其self和this的变化,  
这里有个有趣奇怪的现象:  
**当this传给一个局部对象的属性时,无论对象的嵌套有多深入,都直接指向全局对象window而非直属对象**  

```js
var i = function() {
    var some = "I'S SOME";
    console.log('~~~整个函数i的this')
    console.log(this);

    var iObj = {
        iObjSelf: this,
        con: function() {
            var self = this;
            console.log('~~~函数i里面的一个局部对象iObj的this')
            console.log(this);

            function iObjCon() {
                console.log('~~~在iObj.con里函数iObjCon()的this')
                console.log(this);
                console.log('~~~在iObj.con里函数iObjCon()的self')
                console.log(self);
            };
            iObjCon();
            return;
        },
        conGo: (function() {
            var self = this;
            console.log('~~~iObj.conGo匿名函数的this')
            console.log(this);

            function iObjConGo() {
                console.log('~~~在iObj.conGo里函数iObjConGo()的this')
                console.log(this);
                console.log('~~~在iObj.conGo里函数iObjConGo()的self')
                console.log(self);
            };
            iObjConGo();
            return some;
        })(),
        anoObj: {
            anoObjSelf: this,
            con: function() {
                var self = this;
                console.log('~~~iObj.anoObj.con的this');
                console.log(this);

                function iObjCon() {
                    console.log('~~~在iObj.anoObj.con里函数iObjCon的this')
                    console.log(this);
                    console.log('~~~在iObj.anoObj.con里函数iObjCon的self')
                    console.log(self);
                    console.log('iObj.iObjSelf=this,结果是 ' + iObj.iObjSelf);
                    console.log('iObj.anoObj.anoObjSelf=this,结果是 ' + iObj.anoObj.anoObjSelf);
                    console.log('就算嵌套在很深的对象的函数里,依然能访问i作用域的局部变量some ' + some);
                };
                iObjCon();
                return;
            },
            some: some
        }
    };
    return (function() {
        i.att = 'i.att';
        iObj.con();
        iObj.anoObj.con();
        console.log('iObj.conGo用匿名函数返回i作用域里的局部变量some ' + iObj.conGo);
        console.log('iObj.anoObj.some=some,嵌套对象的属性依然能得到some ' + iObj.anoObj.some);
        return 'done';
    })();
};
```

```js
i()
```
>  
> 整个函数i的this  
> > **Window**  
>  
> iObj.conGo匿名函数的this  
> > **Window**  
>  
> 在iObj.conGo里函数iObjConGo()的this  
> > **Window**  
>  
> 在iObj.conGo里函数iObjConGo()的self  
> > **Window**  
>  
> 函数i里面的一个局部对象iObj的this  
> > **Object {iObjSelf: Window, con: function, conGo: "I'S SOME", anoObj: Object}**  
>  
> 在iObj.con里函数iObjCon()的this  
> > **Window**  
>  
> 在iObj.con里函数iObjCon()的self  
> > **Object {iObjSelf: Window, con: function, conGo: "I'S SOME", anoObj: Object}**  
>  
> iObj.anoObj.con的this  
> > **Object {anoObjSelf: Window, con: function, some: "I'S SOME"}**  
>  
> 在iObj.anoObj.con里函数iObjCon的this  
> > **Window**  
>  
> 在iObj.anoObj.con里函数iObjCon的self  
> > **Object {anoObjSelf: Window, con: function, some: "I'S SOME"}**  
>  
> iObj.iObjSelf=this,结果是  
> > **[object Window]**  
>  
>iObj.anoObj.anoObjSelf=this,结果是  
> > **[object Window]**  
>  
> **就算嵌套在很深的对象的函数里,依然能访问i作用域的局部变量some I'S SOME**  
> **iObj.conGo用匿名函数返回i作用域里的局部变量some I'S SOME**  
> **iObj.anoObj.some=some,嵌套对象的属性依然能得到some I'S SOME**  
> **"done"**  
>  

