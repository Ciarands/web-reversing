# Steps to defeat:
1. Step 1 -
>! Replace the eval with console.log.
2. Step 2
>! Evaluate the code.

```js
(function(p){ec=function($){let e=0;for(let n=0;n<$.length;n++){const t="0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".indexOf($[n]);if(-1===t)return-1;e=62*e+t}return e};const[d,c]=p.split("|"),dc=d.split(","),u=c.replace(/\$\$([0-9a-zA-Z]+)/g,(($,n)=>(e=ec(n),e>=0&&e<dc.length?dc[e]:$)));eval(u)})('const,isAdmin,false,function,validateUser,name,if,return,You,do,not,have,permission,to,access,this,resource,This,incident,will,be,reported,flag,h3110,console,log|\n$$0 $$1 = $$2;\n$$3 $$4($$5) {\n    $$6 (!$$1) $$7 "$$8 $$9 $$a $$b $$c $$d $$e $$f $$g. $$h $$i $$j $$k $$l.";\n    $$7 "$$m{$$n-4dm1n1str4t0r}";\n}\n$$o.$$p($$4());\n');
```