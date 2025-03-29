# Analysing Dean Edwards packer
Dean Edwards Packer is a JavaScript code compression tool that reduces the size of your code by removing unnecessary whitespace and anything reused. 
Although this is arguably not obfuscation, the output is incredibly unreadable, and it's a great starting point to get used to analysing hard-to-read code.

## Let's take a look
Here is an example of packed code:

```js
eval(function(p,a,c,k,e,r){e=function(c){return c.toString(a)};if(!''.replace(/^/,String)){while(c--)r[e(c)]=k[c]||e(c);k=[function(e){return r[e]}];e=function(){return'\\w+'};c=1};while(c--)if(k[c])p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c]);return p}('0 i="9";0 3="e";0 1="a";0 2="b";0 4=", ";0 5="c";0 6="f";0 7="d";0 8="!";g.h(i+3+1+1+2+4+5+2+6+1+7+8)',19,19,'var|iii|iiii|ii|iiiii|iiiiii|iiiiiii|iiiiiiii|iiiiiiiii|H|l|o|W|||r|console|log|'.split('|'),0,{}))
```

This may look intimidating at first glance; let's approach it step by step.

## Beautifying the code
In my experience, human brains are terrible at parsing information all at once, so let's clean it up a little bit to try and reframe what we're looking at.
```js
eval(function(p, a, c, k, e, r) {
  e = function(c) {
    return c.toString(a)
  };
  if (!''.replace(/^/, String)) {
    while (c--) r[e(c)] = k[c] || e(c);
    k = [function(e) {
      return r[e]
    }];
    e = function() {
      return '\\w+'
    };
    c = 1
  };
  while (c--)
    if (k[c]) p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]);
  return p
}('0 i="9";0 3="e";0 1="a";0 2="b";0 4=", ";0 5="c";0 6="f";0 7="d";0 8="!";g.h(i+3+1+1+2+4+5+2+6+1+7+8)', 19, 19, 'var|iii|iiii|ii|iiiii|iiiiii|iiiiiii|iiiiiiii|iiiiiiiii|H|l|o|W|||r|console|log|'.split('|'), 0, {}))
```

Here I have taken the same code and put it into https://beautifier.io/, a JS beautifier.
You may notice it's a simple string replacement mechanism. How can we attack this?

## Attacking the simplest weakness
Before understanding the replacement mechanism, it is always a good idea to target the weakest point when doing your analysis. We know that eval MUST take valid Javascript, so we can mentally filter out most of what we see.
Instead of allowing eval to execute the code, we can intercept it and inspect the output using console.log.

```js
console.log(function(p, a, c, k, e, r) {
  e = function(c) {
    return c.toString(a)
  };
  if (!''.replace(/^/, String)) {
    while (c--) r[e(c)] = k[c] || e(c);
    k = [function(e) {
      return r[e]
    }];
    e = function() {
      return '\\w+'
    };
    c = 1
  };
  while (c--)
    if (k[c]) p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]);
  return p
}('0 i="9";0 3="e";0 1="a";0 2="b";0 4=", ";0 5="c";0 6="f";0 7="d";0 8="!";g.h(i+3+1+1+2+4+5+2+6+1+7+8)', 19, 19, 'var|iii|iiii|ii|iiiii|iiiiii|iiiiiii|iiiiiiii|iiiiiiiii|H|l|o|W|||r|console|log|'.split('|'), 0, {}));
```

Instead of manually deciphering the techniques used, this method allows JavaScript to handle the decoding for us.
In most cases, working smarter rather than harder is the best way to analyse and defeat these problems.
However, there are many instances where you may want to reverse engineer this further, such as in an environment where you are evaluating this in a separate programming language, so let's continue with making some observations.

## Making initial observations
Starting off, we can see there is an anonymous function inside the eval call.
For now, let's ignore everything inside the anonymous function and look at the function arguments. We can see that there are 6 unique args. Let's break down what is being passed:
- p = type("string")
We can see that the p argument looks a little like pseudocode; assignments are being made, and they're later referenced in a "g.h" call.

- a = type("integer")
There isn't much noteworthy about this yet; let's pass over this for now.

- c = type("integer")
Same as a, not particularly noteworthy.

- k = type("array")
This is a little more interesting; here we can see a string being split at each "|" into an array. In this we can see some indexes that stand out, such as "var" and "console".

- e = type("integer")
An integer that is 0; this could be used as an integer or a falsy value in the future.

- d = type("object")
An empty object.

## Understanding the Replacement Mechanism
Now that we have taken a quick glance at what's being passed to the function, let's see what it does with these params.
There is no "one perfect way" to go about this; however, a lot of the time in obfuscated code there is a lot of stuff that we really don't care about, which we can mentally discard. To me, the thing that stands out most is the pseudocode-esque string at the start, which corresponds with the `p` argument, so let's jump to this immediately and work backwards.

```js
while (c--)
  if (k[c]) p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]);
return p
```

We can see that the first reference to `p` is in this while loop, in which a regex pattern is created and used to replace something in our `p` argument with something in our `k` argument, which, if you remember from earlier, was our array with some interesting indexes such as `var` and `console`.
So we can determine that this iterates over the `k` array, replacing encoded tokens in the packed string `p` with their original values.

## Discarding useless stuff
- We can note that `e`, the integer we suggested might be a falsy value, is actually reassigned immediately and then reassigned again in a if statement that will always resolve to `true`.
- `r` will only ever be assigned either `k[c] || e(c)`, discarding will result in the exact same logic as the later while loop.
- We can also discard `c` as it directly corresponds to the length of `k` per the `k[c]`.

From p,a,c,k,e,r to p,a,k :D (packing the packer??)

Alternatively another way of approaching this could be to just entirely delete our if block:
```js
eval(function(p, a, c, k, e, r) {
  e = function(c) {
    return c.toString(a)
  };
  while (c--)
    if (k[c]) p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]);
  return p

}('0 i="9";0 3="e";0 1="a";0 2="b";0 4=", ";0 5="c";0 6="f";0 7="d";0 8="!";g.h(i+3+1+1+2+4+5+2+6+1+7+8)', 19, 19, 'var|iii|iiii|ii|iiiii|iiiiii|iiiiiii|iiiiiiii|iiiiiiiii|H|l|o|W|||r|console|log|'.split('|'), 0, {}))
```
However, this relies on JavaScript's kind of strange `toString` implementation, which supports a base.


## Simplifying and rewriting this
After breaking down this problem, let's reconstruct it in Python.
Since Python doesn't support `toString()` with a radix parameter like JavaScript does, we can approach this similarly to the population of `r` and regex substitute all groups of `\w+`.

```py
import re

def unpack(packed_js):
  matched_data = re.search(r"}\('(.+)',(\d+),\d+,'(.+)'\.split", packed_js)
  if not matched_data:
    return None

  psuedocode = matched_data.group(1)
  base = int(matched_data.group(2))
  variables = matched_data.group(3).split("|")

  def replace_match(match):
    token = match.group(1)
    num = int(token, base)
    return variables[num] or token

  unpacked_js = re.sub(r'(\w+)', replace_match, psuedocode)
  return unpacked_js

if name == "__main__":
  packed_input = """eval(function(p,a,c,k,e,r){e=function(c){return c.toString(a)};if(!''.replace(/^/,String)){while(c--)r[e(c)]=k[c]||e(c);k=[function(e){return r[e]}];e=function(){return'\\w+'};c=1};while(c--)if(k[c])p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c]);return p}('0 i="9";0 3="e";0 1="a";0 2="b";0 4=", ";0 5="c";0 6="f";0 7="d";0 8="!";g.h(i+3+1+1+2+4+5+2+6+1+7+8)',19,19,'var|iii|iiii|ii|iiiii|iiiiii|iiiiiii|iiiiiiii|iiiiiiiii|H|l|o|W|||r|console|log|'.split('|'),0,{}))"""
  unpacked_output = unpack(packed_input)
  print(unpacked_output)
```

## Challenge
Retrieve the flag from this custom-packed script:
```js
(function(p){ec=function($){let e=0;for(let n=0;n<$.length;n++){const t="0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ".indexOf($[n]);if(-1===t)return-1;e=62*e+t}return e};const[d,c]=p.split("|"),dc=d.split(","),u=c.replace(/\$\$([0-9a-zA-Z]+)/g,(($,n)=>(e=ec(n),e>=0&&e<dc.length?dc[e]:$)));eval(u)})('const,isAdmin,false,function,validateUser,name,if,return,You,do,not,have,permission,to,access,this,resource,This,incident,will,be,reported,flag,h3110,console,log|\n$$0 $$1 = $$2;\n$$3 $$4($$5) {\n    $$6 (!$$1) $$7 "$$8 $$9 $$a $$b $$c $$d $$e $$f $$g. $$h $$i $$j $$k $$l.";\n    $$7 "$$m{$$n-4dm1n1str4t0r}";\n}\n$$o.$$p($$4());\n');
```

# Steps to defeat (spoilers):
<details>
  <summary>Step 1</summary>
  
  Replace the eval with console.log.
</details>

<details>
  <summary>Step 2</summary>
  
  Evaluate the code.
</details>