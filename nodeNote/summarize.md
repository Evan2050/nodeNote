##global
```
console.log(this); //在文件中直接访问this 不是global 而是{}

(function () {
 console.log(this);     => global
})();

var a = 100;
global.a = function () {};
console.log(global.a); // => function  直接var的变量不会直接挂载在global上
//特点 可以在全局下任何地点访问

console.log(global);
    - process 进程
    - Buffer 缓存
    - clearImmediate
    - clearInterval: [Function],
    - clearTimeout: [Function],
    - setImmediate: [Function],
    - setInterval: [Function],
    - setTimeout: [Function],
    - console: [Getter]
    
console.error('error'); //输出错误
console.info('info'); //输出信息
console.log('log'); //输出日志
console.warn('warn'); //输出警告 //顺序是不固定
```
##exports(导出)
```
this = module.exports = exports = {}
console.log(this===module.exports);     => true

function Person() {
    this.meat = 'sweet'
}
// exports.Person = Person;
module.exports = Person;
/*
   (function(require,exports,module,__dirname,__filename){
   //1
         module.exports = exports = {}
         return module.exports
   //2
        module.exports和exports使用的是同一个{}
        return module.exports还是原来的地址
   })
*/
```
##setImmediate(一个异步方法，并且后面不能指定时间)
```
setImmediate(function () {
   console.log('setImmediate');
});
console.log('current');
setTimeout(function () {
    console.log('setTimeout');
},20);
//在setTimeout没有给时间的时候 setImmediate是给setTimeout一些机会的
process.nextTick(function () {
    console.log('nextTick'); //服务有两个小本 他是当前小本的底部
});
// nextTick > setImmediate > setTimeout > io
```
##process
```
cwd current working directory
 __dirname  当前文件所在的文件夹的路径,不是global上的属性
 --filename 当前文件绝对路径,不是global上的属性
console.log(process.cwd()); //在哪里执行(可变)
console.log(__dirname);
chdir  change directory
process.chdir('..');  (改变)
console.log(process.cwd());
console.log(__dirname);
```
##module
- 引用模块
```
引用文件采用的是./ 文件模块
第三方模块
核心模块
require是同步的，有回调函数的是异步的，能直接通过返回值拿到内容的是同步的
require('jwnewpack');
它会找node_modules文件夹 找对应的jwnewpack文件名
先找文件里是否包含index.js index.json ,会查找package.json里对应的main字段，找到对应的执行文件，如果没有像上级查找，直到根目录结束，如果没有报错
console.log(module.paths); //查找目录的顺序
```
- 模块缓存
```
var a = require('./cache.js');
delete require.cache[require.resolve('./cache.js')];//模块缓存的文件是可以删除的
var a = require('./cache.js');
//多次引用同一个模块不会被重复加载
//会将加载的模块进行缓存
console.log(require["cache"]); //根据绝对地址来缓存
//想删除缓存，构建一个绝对路径，通过路径去对象中查找，删除掉对应的模块

//通过名字,构造出一个绝对路径来
console.log(require.resolve('./cache.js'));
//在去对象中删除
```
##util
```
// util是node自带的核心包，不需要安装直接require即可
var util = require('util');

1.继承的方法
util.inherits();

2.原型继承
function Parent() {
    this.meat = 'sweet';
}
Parent.prototype.handsome = function () {
    console.log('很帅');
}
function Child() {}
Child.prototype = new Parent();
原型链继承
Child.prototype.__proto__ = Parent.prototype;
Child.prototype = Object.create(Parent.prototype);
Object.setPrototypeOf = function (cprop,superprop) {
    cprop.__proto__ = superprop;
    return {};
};
Object.setPrototypeOf(Child.prototype,Parent.prototype);
util.inherits(Child,Parent); //子类继承父类的原型上的方法
var child = new Child();


//util.inspect解析方法 console.dir默认调用的方法
var school = {name:'zfpx',value:{value:1}};
//定义不可枚举的属性
Object.defineProperty(school,'age',{ //给对象定义属性
    enumerable:false, //是否可枚举
    value:8, //属性的值
    writable:false,//是否可写(改变值)
    configurable:true //是否可配置(删除)
});
console.log(school);
/* legacy: obj, showHidden 显示隐藏属性, depth 深度, colors 颜色*/
console.log(util.inspect(school,{showHidden:true,depth:1,colors:true}));

```
- 检测数据类型
```
util.isArray([]);
util.isBoolean([]);
util.isDate(new Date());
```

##events(核心模块)
- 事件绑定 on
- 绑定一次 once
- 触发事件 emit
- 移除事件 removeListener
- 移除所有事件 removeAllListener
```
var EventEmitter = require('events');
var event = new EventEmitter();
function eat(who) {
    console.log(who+'饿了');
}
function drink(who) {
    console.log(who+'喝水');
}
event.once('fn',eat);
//event.removeListener('饿了',eat);
//event.removeAllListeners('饿了');//移除所有事件
event.emit('fn','我');
event.emit('fn','我');

//注释:node最大的绑定数目默认是10个超出会有异常
```

##buffer
用来进制计算，一个字节有八个位组成，每个位最大时1

- 计算：当前位的最大值*进制^所在第几位
    - eg：00000011是二进制 转化成10进制  1*2^1+1*2^0 = 3
    - ff 16进制最大  16进制转换成10进制  15*16^1+15*16^0 = 255
    
- NODE中的buffer
    - 创建buffer
        * 通过数字的方式来创建buffer：var buffer = new Buffer(4);【将buffer写成固定的内容buffer.fill(1);】
        * 通过数组的方式创建buffer：var buffer = new Buffer([1,2,3]);
        * 字符串创建：var buffer = new Buffer('珠','utf8');【在utf8中为一个汉字三个字节】   
    - buffer与字符串的区别
        * 字符串具有不变性
            ```
            var str = 'ab';
            str[0] = 'c';
            console.log(str);  //ab
            ```
        * 字符串默认为字符的长度，buffer为字节的长度    
    - buffer和数组的区别
        都有slice方法，操作数组原有数组不改变，而buffer任然操作的时原buffer
    - buffer中的方法
        * write(string,offset,length,encoding)
        ```
        buffer.write('珠峰',0);//默认写入长度为内容长度,编码格式utf8
        buffer.write('培训',6,6,'utf8'); //数组里的方法包前不包后 arr.slice(0,1)
        console.log(buffer.toString()); //将buffer转化为字符串
        ```
        * copy方法(targetBuffer,targetStart,sourceStart,sourceEnd ) 将n个小buffer粘贴到buffer上
        ```
        var buffer = new Buffer(12);
        var buf1 = new Buffer('珠峰培');
        var buf2 = new Buffer('训');
        buf1.copy(buffer,0);  //默认copy原的0到结束
        buf2.copy(buffer,9);
        console.log(buffer.toString()); //copy方法将多个小buffer写入到大buffer上
        ```
        * concat(arr,totalLength)方法 将多个buffer拼成一个大buffer
        ```
        var buf1 = new Buffer('学习');
        var buf2 = new Buffer('前端');
        var buf3 = new Buffer('NODE');
        var newBuffer = Buffer.concat([buf1,buf2,buf3]);
        console.log(newBuffer.toString);
        //自己写一个myConcat方法，判断是否传入长度,长度过大，留取有效长度
        Buffer.myConcat = function(list,totalLength){
            //获取buf1和buf2 将两个buffer粘贴到大buffer（根据长度构建buffer）
            //截取有效的buffer返回
            //先判断是否传递 totalLength
            if(typeof totalLength == 'undefined'){
                totalLength = 0;
                list.forEach(function(buf){
                    totalLength += buf.length;
                })
            }
            //通过长度来构建一个大buffer
            var buffer = new Buffer(totalLength);
            //目的是将每一个小buffer copy到大buffer上
            var index = 0;  //存储偏移量
            list.forEach(function(buf){
                buf.copt(Buffer,index);
                index += buf.length;
            })
            return Buffer.slice(0,index); //防止totalLength过大
        }
        console.log(Buffer.myConcat([buf1,buf2,buf3]).toString());
        ```
        * isBuffer() 检验是否为buffer
        ```
        console.log(Buffer.isBuffer([1,2,3]));  //false
        ```
    - base64(编码格式,是将内容转换成可见编码)
        * 取值（64）
            ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/
        * 进制转化
           ```
           1.任意进制转换成10进制
           console.log(parseInt('11111111',2));  
           
           2.将任意进制转换成任意进制
           console.log((0xff).toString(2)); //默认10进制，可以指定转化的进制
           console.log((100).toString(16));
           ```
        * 一个简单的方法，输入任意汉字，转换出base64编码
        ```
        //1.先将buffer里的每一个字节全部转换成2进制，拼接到一起
        //3.101010 101010 10每6位 分开，前面+ 两个0
        //3.转换成10进制
        //4.去可见编码中取值
        function base64(str){
            var buffer = new Biffer(str);
            var str1 = '';
            buffer.forEach(function(buf){
                str1 += (buf).toString(2);  //将10进制转换成2进制，累加
            })
            var arr = [];
            for(var i=0; i<str1.length/6; i++){
                arr.push(parseInt('00'+str1.slice(i*6,(i+1)*6),2));  //没6位分开，补00转化成10进制
            }
            var str2 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/';
            var BASE = '';
            arr.forEach(function(item){
                BASE += str2[item];
            })
            return BASE;
        }
        console.log(base64('淡定'));
        ```
##fs(核心模块)
- readFile和readFileSync(读)
    ```
    var fs = require('fs');
    同步用try catch捕获异常
    try{
        var data = fs.readFileSync('.name.txt','utf8');
    }catch(e){
        console.log(e);
    }
    //'r' read读取 'w' 写  'a'  追加
    //encoding默认编码格式是null buffer类型
    异步 不会阻塞主线程
    fs.readFile('./name.txt','utf8',function(err,data){
        if(err)console.log(err);    //err为错误信息，参数不可变化
        console.log(data);
    })
    ```
    
    ```
    var fs = require('fs');
    var person = {};
    fs.readFile('./name.txt','utf8',function(err,data){
        person.name = data;  //读取成功之后的回调
        out();
    })
    fs.readFile('./age.txt','utf8',function(err,data){
        person.age = data;
        out();
    })
    function out(){
        //Object.keys(person)对象属性名组成的数组  Object.values(person)对象属性值组成的数组
        if(Object.keys(person).length == 2){   //说明两个已经读取完成
            console.log(person);
        }
    }
    ```
    
    ```
    var fs = require('fs');
    var EventEmitter = require('events');
    var event = new EventEmitter();
    var person = {};
    fs.readFile('./name.txt','utf8',function(err,data){
        person.name = data;
        event.emit('输出')；
    })
    fs.readFile('./age.txt','utf8',function(err,data){
        person.age = data;
        event.emit('输出')；
    })
    function out(){
        if(Object.keys(person).length == 2){
            console.log(person);
        }
    }
    event.on('输出',out);
    ```
- writeFile和writeFileSync(写)
    ```
    第一个参数：写入的路径；第二个参数：写入的内容默认时utf8
    var fs = require('fs');
    fs.writeFileSync('./name1.txt',new Buffer('自习'));
    fs.writeFile('./name1.txt',new Buffer('自习'),function(){
        //成功后的回调，当写入完成时执行的函数
    });
    ```
    
    ```
    //写一个拷贝文件的方法 同步的异步的, 先读取后写入
    var fs = require('fs');
    function copySync(source,target){
        var result = fs.readFileSync(source,'utf8');
        fs.writeFileSync(tarfet,result);
    }
    copySync('./name.txt','./name1.txt');
    function copy(source,target){
        fs.readFile(source,'utf8',function(err,data){
            if(err)console.log(err);
            fs.writeFile(target,data);
        })
    }
    copy('./name.txt','./name1.txt');
    //readFile的缺点，不能读取比内存大的文件，适合读小文件（读取到内存中的）
    ```
- appendFile和appendFileSync(追加文件内容)
用法与writeFile和writeFileSync一样
```
var fs = require('fs');
fs.appendFile();
fs.appendFileSync();
```
- mkdirSync和mkdir(创建文件夹)
```
var fs = require('fs');
参数(path,mode)  mode:目录权限（读写权限），默认0777
fs.mkdirSync('a'); //必须保证上一级存在才能创建下一级
fs.mkdir('a',function(err){     //如果目录已存在，将抛出异常。
    if(err){
        console.log(err);
    }else{
        console.log("creat done!");
    }
})
```
- existsSync和exists(判断文件夹是否存在)
```
var fs = require('fs');
var flag = fs.existsSync('a');  //a文件夹存在返回true，否则flag为false
fs.exists('a',function(exists){
    console.log(exists);    //a文件夹存在返回true，否则false
})
```
```
同步创建文件夹a/b/c/d
var fs = require('fs');
function makeSyncpPp){
     //先创建a 在创建a/b  在创建a/b/c 在创建 a/b/c/d
     var arr = p.split('/');    //[a,b,c,d]
     for(var i=0; i<arr.length; i++){
        var path = arr.slice(0,i+1);
        if(!fs.existsSync(path){
            fs.mkdirSync(path);
        }
     }
}
makeSyncpPp('a/b/c/d');
```
```
异步创建文件夹a/b/c/d（fs.mkdir + fs.exists 递归）
var fs = require('fs');
function makeP(p){
    
}
makeP('a/b/c/d');
```
- statSync和stat
会判断一些文件的状态，文件是否有更新（是否修改过）,文件的出生时间，文件的访问的时间
```
参数[path]
var fs = require('fs');
var statInfo = fs.statSync('./name.js');
console.log(statInfo);
/*
atime: 2016-11-05T10:04:52.232Z, access time 访问的时间
mtime: 2016-11-05T10:04:52.233Z, modify time 修改的时间  更改过但是但是不一定有修改
ctime: 2016-11-05T10:04:52.239Z, change time 修改的时间  内容一定要发生变化
birthtime: 2016-11-05T09:48:23.053Z birth time 出生时间
*/
fs.stat('./kong.js',function(err,stats){
    if(err){
        console.log(err);
    }else{
        console.log(stats);
    }
})
```
```
var fs = require('fs');
var stat = fs.statSync('./kong.js');
console.log(stat.isFile());  //判断是否为文件
console.log(stat.isDirectory());  //判断是否时文件夹
```
##path(核心模块，专门处理路径的一个模块)
- basename(获取不带后缀的文件名,可以指定扩展名留取想要的部分)
```
var path = require('path');
console.log(path.basename('1.min.js','.min.js'));  // => 1
```
- extname(获取文件的扩展名, 读取最后一个.后的内容)
```
var path = require('path');
console.log(path.extname('1.min.js'));  // => .js
```
- join(连接路径)
```
var path = require('path');
以当前文件解析出一个绝对路径
path.join(__dirname,'a/../b');  // => ../表示上一级  __dirname当前文件所在的文件夹
```
- resolve(解析路径)
resolve(from,to)
from    源路径(默认为当前文件所在文件夹的路径)
to      将被解析到绝对路径的字符串
```
var path = require('path');
path.resolve();    => F:\zhufeng\zhufeng\NODE\summarize
path.resolve('../../100.js');   => F:\zhufeng\zhufeng\100.js
path.resolve('/foo/bar', './baz');    => F:\foo\bar\baz
```
##stream(流是从上到下，始终是一个)
- createReadStream(创建可读流)[data，end，error，pause，resume]
```
var fs = require('fs');
var rs = fs.createReadStream('./name.txt',{highWaterMark:1,start:3,end:5}); //包前又包后
/*
    rs不是读取到的数据,而是一个可读流的对象
    相当在家里安了一个水管，默认是非流动模式
    监听on data事件将水管打开，fs会以最快的速度不停的发射emit data事件
    encoding:null 类型为buffer
    highWaterMark 64*1024 默认读取64k
*/
var arr = [];
//从非流动模式->流动模式
rs.on('data',function(data){
    //data是我们需要的数据
    arr.push(data);
})
//监听结束
rs.on('end',function(){
    console.log(Buffer.concat(arr).toStrinf());
})
//监听错误
rs.on('error',function(err){
    console.log(err);
})
```
```
var fs = require('fs');
var rs = fs.createReadStream('./name.txt',{highWaterMark:3});
rs.setEncoding('utf8'); //如果编码格式是utf8 最小水位线就是3
var str = '';
rs.on('data',function(data){    //默认每次读取64k 是怎么读的 => 不停的以最快速度将内容读取到内存中
    str += data;
    rs.pause(); //停止触发data事件
})
var timer = setInterval(function(){
    rs.resume();恢复触发data事件
},1000)
rs.on('end',function(){
    console.log(str);
    clearInterval(timer);
})
```
- createWriteStream(创建可写流)[write，end，drain，pipe]
```
var fs = require('fs');
var ws = fs.createWriteStream('./name1.txt',{highWaterMark:1});
/*
    ws为可写流对象
    highWaterMark 16384 = 16*1024  按理说应该  如果无法写入就不要在读取了，等我都写入完了，再去读   
    write end方法 (response)
*/
var flag = ws.write('1');   //必须是字符串或者buffer类型  ==>返回flag，表示此次之后还是否可写
var flag = ws.write('2'); //这个时候已经饱了
ws.end();   //一旦调用end方法，就会把所有地下的吃的全捡起来，并且把end中的内容，一起吃到肚子里，把嘴合上，end后无法调用write
```
```
var fs = require('fs');
var ws = fs.createWriteStream('./name1.txt',{highWaterMark:5});
//写10个数，每次要保证写入1个后，在写入一个
var index = 0;
function w(){
    var flag = true;
    //最大不能超过10  并且flag不能为false
    while(flag && index<10){
        flag = ws.write(''+index++);
    }
}
w();
ws.on('drain',function(){
    console.log('可以写入了');
    w();
});
//写内存相当于一个嘴巴，这个嘴里只能装2个馒头，妈妈碗里有10个，喂到我第二个的时候，告诉他我吃不下了，等我消化后也就是drain方法，告诉他继续喂我吧
```
```
pipe();原理
//创建一个可读流 和一个可写流
//1.每次读入一次后开始写入 在on data 中调用wirte方法
//2.当文件不能写入时 暂停读取 如果写入返回false 调用pause
//3.等待抽干后再调用恢复方法，恢复读取 在drain方法中调用resume
//4.监听读取后关闭掉写入方法ws.end方法 rs.on('end')方法中调用ws.end
var fs = require('fs');
function pipe(source,target){
    var rs = fs.createReadStream(source);
    var ws = fs.createWriteStream(target);
    rs.on('data',function(data){
        var flag = ws.write(data);
        if(!flag){rs.pasue();}
    });
    ws.on('drain',function(){
        rs.resume();
    })
    rs.on('end',function(){
        ws.end();
    })
}
pipe('./name.txt','./name1.txt');
```

```
var fs = require('fs');
function copy(source,target) {
    var rs = fs.createReadStream(source);
    var ws = fs.createWriteStream(target);
    rs.pipe(ws); //将可读流中的内容，导入到可写流中
}
copy('./name.txt','./name1.txt');
```
## http
- server
```
var http = require('http');
var fs = require('fs');
var path = require('path');
var url = require('url');
var mime = require('mime');  //第三方模块
http.createServer(function(req,res){
    //req客户端的请求
    //res是服务端的响应  res是个流 可写流  req是个流 可读流
    //res.writeHead(200,{'Content-Type':'text/plain;charset=utf8'});
    //statusCode + setHeader = writeHead
    //要根据请求的内容返回不同的结果  路由
    console.log(req.url); //默认跟路径是/
    console.log(req.method); //请求的方法
    console.log(req.headers); //获取所有请求头,通过小写的属性名来获取内容
    var pathname = url.parse(req.url,true).pathname;
    if(pathname == '/'){
        res.setHeader('Content-Type','text/html;charset=utf8');
        fs.createStream('./index.html').piep(res);
    }else{
        fs.exists('.'+pathname,function(exists){
            if(exists){
                //查看当前路径的文件类型
                res.createStream('Content-Type',mime.lookup(pathname)+';charset=utf8');
                fs.createStream('.'+pathname).pipe(res);
            }else{
                res.statusCode = 404;
                res.end('not found');
            }
        })
    }
}).listen(3000);
```
- 缓存
```
根据修改时间来判断
var http = require('http');
var fs = require('fs');
http.createServer(function (req,res) {
    if(req.url == '/'){
        res.setHeader('Content-Type','text/html;charset=utf8');
        fs.createReadStream('./index.html').pipe(res);
    }else if(req.url == '/index.js'){
        var timer = fs.statSync('./index.js').ctime.toUTCString();
        var ctime = req.headers['if-modified-since']; //上一次修改时间
        if(ctime&&(timer==ctime)){ //缓存的时间和当前修改过的时间相同，才会调用缓存
            res.statusCode = 304;
            res.end('');
        }else{
            res.setHeader('Last-Modified',timer);
            fs.createReadStream('./index.js').pipe(res);
        }
    }else{
        res.statusCode = 404;
        res.end('not found');
    }
}).listen(8888);
```

```
将内容通过加密发送给浏览器，客户端再次请求时，判断内容是否改变
var http = require('http');
var fs = require('fs');
var crypto = require('crypto'); //node的加密方法
http.createServer(function (req,res) {
    if(req.url == '/'){
        res.setHeader('Content-Type','text/html;charset=utf8');
        fs.createReadStream('./index.html').pipe(res);
    }else if(req.url == '/index.js'){
        //内容频繁的更改，精确不到秒，很多服务器修改时间不准确
        //判断文件中的内容，比较上一次的内容，和这一次的内容，如果有区别说明更改过，发送一个最新的内容，给浏览器端
        var data = fs.readFileSync('./index.js');
        data = crypto.createHash('md5').update(data).digest('base64');  //加密
        console.log(data); //当前 ***
        //都能获取一个最新值和上次设置的比  if-none-match
        var match = req.headers['if-none-match']; //上一次  ***
        if(match&&(match==data)){
            res.statusCode =304;
            res.end('');
        }else{
            res.setHeader('Etag',data);//第一次访问时把内容进行加密放到头部
            fs.createReadStream('./index.js').pipe(res);
        }
    }else{
        res.statusCode = 404;
        res.end('not found');
    }
}).listen(3000);
```

```
设置缓存时间，如果在缓存时间内不用发送任何请求
var http = require('http');
var fs = require('fs');
var crypto = require('crypto');
http.createServer(function (req,res) {
    if(req.url == '/'){
        res.setHeader('Content-Type','text/html;charset=utf8');
        fs.createReadStream('./index.html').pipe(res);
    }else if(req.url == '/index.js'){
        console.log(1);
        //如果在缓存时间内不用发送任何请求
         res.setHeader('Expires',new Date(new Date()+3000).toUTCString());
         res.setHeader('Cache-control','max-age=3');   //单位是秒
         fs.createReadStream('./index.js').pipe(res);
    }else{
        res.statusCode = 404;
        res.end('not found');
    }
}).listen(3000);
```