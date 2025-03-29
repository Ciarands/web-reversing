# Intro to transformation obfuscation / source uglification
Transformation obfuscation/source uglification is a particular subset of obfuscation in which we only really modify the structure of the code; without taking advantage of virtualisation or similar techniques.
Although this might sound relatively simplistic, when exploiting language-specific quirks, this can result in some reasonably hard to analyse code.
Modern web obfuscation frequently combines multiple transformation layers which you may already be familiar with, such as:
- Initial minification (Webpack, UglifyJS)
- Custom string encoding/rotation
- Control flow flattening
- Environmental checks (detect DevTools)

## JavaScript is amazing for transformation obfuscation...
Consider this simple function:
```js
function validate(password) {
    return password === 'secret';
}
```

Here is a transformed counterpart:
```js
const _0x3a9f=['\x73\x65\x63\x72\x65\x74'];(function(_0x1,_0x2){const _0x3=function(_0x4){while(--_0x4){_0x1['push'](_0x1['shift']());}};_0x3(++_0x2);}(_0x3a9f,0x1f3));const _0x1342=function(_0x1,_0x2){_0x1=_0x1-0x0;let _0x3=_0x3a9f[_0x1];return _0x3;};function validate(_0x4){return _0x4===_0x1342('0x0');}
```

This transformation uses three key techniques:
- Hex Escape Encoding: '\x73\x65\x63\x72\x65\x74' represents 'secret'.
- Array Shuffling: Self-modifying array that reorganizes its elements.
- Proxy Functions: _0x1342 acts as indirection layer for string lookup.

As you can see this is already reasonably unreadable, and we can do so much better.

## Why is transformation obfuscation is so common?
Web-Specific Advantages:
- Works with browser's native parser
- Easy to implement with build tools (Webpack, Babel)

Effective Against:
- Casual code inspection
- Basic scraping attempts

Development Cost:
- Often cheaper to implement than VM-based solutions
- Can be entirely automated with existing tools

## A few examples of possible transformation techniques
### 1. String Splitting and Concatenation
> Original:
```js
const API_ENDPOINT = 'https://api.example.com/v1/login';
```
> Obfuscated:
```js
const _0xad = ['htt','v1','api.ex','.c','in', 'ps:','ample', '/','om','log'].reverse();
const API_ENDPOINT = _0xad[9] + _0xad[4] + _0xad[2] + _0xad[2] + _0xad[7] + _0xad[3] + _0xad[6] + _0xad[1] + _0xad[2] + _0xad[8] + _0xad[2] + _0xad[0] + _0xad[5];
```

### 2. Deadcode injection
```js
function processData(data) {
  if (false) { console.log('Never executed!'); } 
  const _temp = [1,2,3].map(x => x*2);
  return data.trim().toLowerCase();
}
```

### 3. Convoluting arbitrary code
Lets say we wanted to load the following function via an EventListenter.
```js
const initApp = function() { console.log("Hello world!") };
```

> Original:
```js
window.addEventListener('load', initApp);
```

> Obfuscated:
```js
[]["filter"]["constructor"]("return this")()['addEventListener'].apply(
  document['defaultView'], 
  ['load'.split().sort(() => 0.5-Math.random()), 
  Function('return '+ ['init','App'].join(''))()]
);
```

### 4. Proxying functions
> Original:
```js
function greet(user) {
  console.log("Hello " + user + "!")
}

greet("user")
```

> Obfuscated:
```js
function _0x12345(a, b, c) {
  return b + a + c; 
}

function _0x1234(a) {
  console.log(a);
}

function _0x123(a) {
  _0x1234(
    _0x12345(
      a,
      "Hello ",
      "!"
    )
  );
}

_0x123("user");
```

## Deobfuscation via analysis
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
/// TODO /// EXPAND ON THIS /// TODO ///
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

Static Analysis:
- Use beautifiers (JSBeautifier, Prettier)
- Search for string operations followed by eval/Function
- Mentally noting things that "stand out"

Dynamic Analysis:
- Use browser debuggers to set breakpoints after string operations
- Hook into Function constructor and eval calls
- Monitor network requests triggered by decoded URLs

## Importance of pattern recognition
Whether playing chess or seeing faces in clouds, pattern recognition is actually pretty helpful.
A real example:
```js
function l() {
  var _ = q;
  var e = [arguments];
  e[2] = [];
  e[1] = 0;
  e[8] = 0;
  e[7] = _.J(11);
  e[8] = 0;
  for (; e[8] < 256; e[8]++) {
    e[2][e[8]] = e[8];
  }
  for (e[8] = 0; e[8] < 256; e[8]++) {
    e[9] = _.J(4);
    e[9] += _.f(8);
    e[9] += 'ngt';
    e[9] += _.f(21);
    e[6] = 'c';
    e[6] += _.J(15);
    e[6] += 'deA';
    e[6] += 't';
    _.l3(0);
    e[5] = _.H8(12, 26624, 7, 12, 3840);
    e[1] = (e[1] + e[2][e[8]] + e[0][0][e[6]](e[8] % e[0][0][e[9]])) % e[5];
    e[3] = e[2][e[8]];
    e[2][e[8]] = e[2][e[1]];
    e[2][e[1]] = e[3];
  }
  e[8] = 0;
  e[1] = 0;
  e[26] = 0;
  for (; e[26] < e[0][1][_.J(2)]; e[26]++) {
    e[4] = 'fromCh';
    e[4] += _.J(22);
    e[4] += _.f(1);
    _.j9(1);
    e[8] = _.g9(256, e[8], 1);
    _.l3(2);
    e[18] = _.H8(239, 6, 1178);
    e[1] = (e[1] + e[2][e[8]]) % e[18];
    e[3] = e[2][e[8]];
    e[2][e[8]] = e[2][e[1]];
    e[2][e[1]] = e[3];
    e[7] += o7[e[4]](e[0][1][_.f(6)](e[26]) ^ e[2][(e[2][e[8]] + e[2][e[1]]) % 256]);
  }
  _.m0();
  return e[7];
}
```
This is a snippet from a real webapp I was reversing about a year ago.
Despite how convoluted this might look, I was able to immediately determine that this was RC4 due to one thing; those two 256 loops.
```
for i from 0 to 255
    S[i] := i
endfor
j := 0
for i from 0 to 255
    j := (j + S[i] + key[i mod keylength]) mod 256
    swap values of S[i] and S[j]
endfor
```
RC4 isnt the strongest but its frequently used due to its speed (used in obfuscator.io, ect).
This ability is best refined through experience, even simple challenges will have you making connections you might have not considered if you hadn't already attempted something similar in the past.

It may sound like generic advice, but practice really does make perfect.

## Challenge
Retrieve the flag from this custom transformation obfuscated script (Bonus points for reconstructing the decryption algorithm):
```js
function _0x5630f6f4f99af80a(_0x7a1bcd3de8b2334c) {
  function _0xcfa51fc73b336843(_0x2255f32f4ff14352, _0x4f396b458f9b5f54) {
    function _0x7766fa34882004d5(_0x9afca74521ed1ff2, s = []) {
      if (_0x9afca74521ed1ff2 >= 256) return s;
      return _0x7766fa34882004d5((function _0x753a2731e5fce723() {
        return _0x9afca74521ed1ff2 + 1;
      })(), [...s, _0x9afca74521ed1ff2]);
    }
    function _0x4918774d6ca87b5c(_0xb2abd31dae6bc255, _0x7715d7192b6213a4, i = +[], j = 0) {
      if (i >= 256) return _0xb2abd31dae6bc255;
      const _0xed4dbe51bebbafb8 = _0x7715d7192b6213a4.length;
      j = (function _0xb81bd24471f655ab() {
        return (j + _0xb2abd31dae6bc255[i] + _0x7715d7192b6213a4[i % _0xed4dbe51bebbafb8]) % 256;
      })();
      [_0xb2abd31dae6bc255[i], _0xb2abd31dae6bc255[j]] = [_0xb2abd31dae6bc255[j], _0xb2abd31dae6bc255[i]];
      return _0x4918774d6ca87b5c(_0xb2abd31dae6bc255, _0x7715d7192b6213a4, (function _0x7791364579ce65c6() {
        return i + 1;
      })(), j);
    }
    function _0x15c295b41e50dcee(_0xbf5211ba79f1f6e0, _0xa5085c1fb8349a07, _0xa9d72d677224107a, _0xa0c4fd58181b9612, decrypted = []) {
      if (_0xa0c4fd58181b9612.length === 0) return new Uint8Array(decrypted);
      _0xa5085c1fb8349a07 = (function _0x9cf00b2b49156c03() {
        return (_0xa5085c1fb8349a07 + 2) % 256;
      })();
      _0xa9d72d677224107a = (function _0x96b4ef55ecc08410() {
        return (_0xa9d72d677224107a + _0xbf5211ba79f1f6e0[_0xa5085c1fb8349a07]) % 256;
      })();
      [_0xbf5211ba79f1f6e0[_0xa5085c1fb8349a07], _0xbf5211ba79f1f6e0[_0xa9d72d677224107a]] = [_0xbf5211ba79f1f6e0[_0xa9d72d677224107a], _0xbf5211ba79f1f6e0[_0xa5085c1fb8349a07]];
      const _0x379bfa7bf91cde91 = _0xbf5211ba79f1f6e0[(function _0x4f244bdcdfc90461() {
        return (_0xbf5211ba79f1f6e0[_0xa5085c1fb8349a07] + _0xbf5211ba79f1f6e0[_0xa9d72d677224107a]) % 256;
      })()];
      const _0xf170ab130ec99fda = _0xa0c4fd58181b9612[0] ^ _0x379bfa7bf91cde91;
      return _0x15c295b41e50dcee(_0xbf5211ba79f1f6e0, _0xa5085c1fb8349a07, _0xa9d72d677224107a, _0xa0c4fd58181b9612.slice(1), [...decrypted, _0xf170ab130ec99fda]);
    }
    let _0x9ab0b44991171008 = typeof _0x4f396b458f9b5f54 === 'string' ? Array.from(_0x4f396b458f9b5f54).map(c => c.charCodeAt(0)) : Array.from(_0x4f396b458f9b5f54);
    const _0x48f13646dc57b677 = _0x7766fa34882004d5(0);
    const _0x218ed5e7fe52f5d4 = _0x4918774d6ca87b5c([..._0x48f13646dc57b677], _0x9ab0b44991171008);
    return _0x15c295b41e50dcee([..._0x218ed5e7fe52f5d4], +[], +[], Array.from(_0x2255f32f4ff14352));
  }
  const _0x728b2a373d4e3416 = "x1794f9eb004adc02";
  const _0xed19237cff75f052 = _0x7a1bcd3de8b2334c.split(/\\x/g).slice(1).map(b => parseInt(b, 16));
  const _0xcbe6c4a8d2aa035d = new Uint8Array(_0xcfa51fc73b336843(_0xed19237cff75f052, _0x728b2a373d4e3416));
  return String.fromCharCode(..._0xcbe6c4a8d2aa035d).replace(/[a-zA-Z]/g, c => String.fromCharCode((c <= 'Z' ? 90 : 122) >= (c = (function _0x8b577d5bba8273c6() {
    return c.charCodeAt(0) + 13;
  })()) ? c : (function _0x8c1a9f993de88fac() {
    return c - 26;
  })()));
}
const _0x0dd8010ea0ddd648 = _0x5630f6f4f99af80a("\\x6a\\x7f\\x95\\xc4");
const _0x66f4b03bcea1dcab = [][_0x5630f6f4f99af80a("\\x71\\x6f\\x9e\\xc6\\x0b\\x67")][_0x5630f6f4f99af80a("\\x72\\x7b\\x86\\xc7\\x1e\\x67\\xbb\\xa5\\x5d\\x9b\\x2a")];
let _0x90879ff77e9d0cf7 = !0;
let _0x9492fac53dc39d58 = [+[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a(""), +[], +_0x5630f6f4f99af80a("")];
function _0x2a538a5c7f96f75f() {
  const _0x2184a13e2a9b1779 = _0x5630f6f4f99af80a("\\x7b\\x6b\\x86\\xd5\\x1e\\x77");
  const _0x81d546f2125c92eb = [(function _0x84bb14462bd90b35() {
    return 34 * 3;
  })(), (function _0xa37de94c04898618() {
    return 12 * 9;
  })(), 97, 103, 123, (function _0xa9a5e11c91c5d615() {
    return 21 * 4;
  })(), (function _0x1873422502ee6a24() {
    return 42 + 40;
  })(), 52, 78, 83, 70, (function _0x90846642d316de02() {
    return 97 % 50 + !_0x90879ff77e9d0cf7;
  })(), 82, 77, 52, (function _0xae7e304d6ce470aa() {
    return 21 * 4;
  })(), 49, 48, 78, (function _0x46a814f093fd1c89() {
    return 9 * 5;
  })(), 49, 83, (function _0xcca5ae48bdd78e60() {
    return 9 * 5;
  })(), 67, (function _0x89c9ca07065a7506() {
    return 97 % 50 + !_0x90879ff77e9d0cf7;
  })(), 48, 76, 33, (function _0x6dbd8251d5dadc5d() {
    return 25 * 5;
  })()];
  let _0x91b5a6eb58206027 = +[];
  const _0x98b150c918ecd697 = {
    next: () => _0x81d546f2125c92eb[(function _0xf5a2aff97233191b() {
      return _0x91b5a6eb58206027++ % _0x81d546f2125c92eb[_0x2184a13e2a9b1779];
    })()]
  };
  _0x9492fac53dc39d58 = _0x9492fac53dc39d58[_0x5630f6f4f99af80a("\\x78\\x77\\x84")](() => _0x98b150c918ecd697[_0x5630f6f4f99af80a("\\x63\\x6b\\x8c\\xc6")]());
}
const _0x50fbaa7b484d5c2b = () => {
  return _0x2a538a5c7f96f75f();
};
function _0x9e84d2778ba8cf36() {
  let _0xb2e066f90b9752fa = [_0x66f4b03bcea1dcab(_0x5630f6f4f99af80a("\\x67\\x6b\\x80\\xc9\\x1c\\x63\\xf3\\xbb\\x5f\\x8d\\x27\\x3d\\xa5\\x46\\x31\\x73\\x26\\x7a\\xea\\xad\\xeb\\xb7\\x8b\\x35\\x42\\x81\\xc9\\x48\\x17\\x29\\x4f\\xe8\\x65\\xbe\\x7c\\x4a\\x8f\\x7b\\x3d\\x6b"))()];
  _0x90879ff77e9d0cf7 = !_0x90879ff77e9d0cf7;
  if (_0xb2e066f90b9752fa[+_0x5630f6f4f99af80a("")][0] !== _0x5630f6f4f99af80a("\\x6c\\x68\\x9d\\xd7\\x18")) {
    throw new Error(_0x5630f6f4f99af80a("\\x4e\\x7b\\x8f\\x81\\x08\\x60\\xf3\\xb4\\x58\\x9e\\x6f\\x32\\xb9\\x4e\\x24\\x35\\x66\\x7a\\xf4\\xa6\\xf4\\xa3\\xd9\\x7c\\x50\\xcf\\xd7\\x44\\x13\\x2a\\x41\\xe3\\x72\\xac\\x76\\x58\\x98\\x6a\\x28\\x2c"));
  }
  _0x50fbaa7b484d5c2b();
}
try {
  console[_0x5630f6f4f99af80a("\\x7b\\x7b\\x93")](_0x9e84d2778ba8cf36(_0x0dd8010ea0ddd648));
  _0x66f4b03bcea1dcab(_0x5630f6f4f99af80a("\\x67\\x6b\\x80\\xc9\\x1c\\x63\\xf3\\xa5\\x58\\x98\\x29\\x25\\xae\\x55"))()[_0x5630f6f4f99af80a("\\x7b\\x7b\\x93")](_0x5630f6f4f99af80a("\\x51\\x60\\x89\\xd5\\x43"), _0x66f4b03bcea1dcab(_0x5630f6f4f99af80a("\\x67\\x6b\\x80\\xc9\\x1c\\x63\\xf3\\x93\\x5d\\x9c\\x39\\x26\\xa3"))()[_0x5630f6f4f99af80a("\\x71\\x7c\\x85\\xdb\\x29\\x77\\xbd\\xb0\\x6a\\x9b\\x3e\\x35")](..._0x9492fac53dc39d58));
} catch (e) {
  console[_0x5630f6f4f99af80a("\\x70\\x7c\\x82\\xc3\\x1c")](e.message);
}
```

# Steps to defeat (spoilers):

<details>
  <summary>Step 1</summary>
  
  Get all decryption function calls through AST parsing or regex `/_0x5630f6f4f99af80a\("([^"]+|)"\)/gm`.
</details>

<details>
  <summary>Step 2</summary>
  
  Replace all encrypted strings with their decrypted counterpart.
</details>

<details>
  <summary>Step 3</summary>
  
  Apply patch to generate flag.
</details>

<details>
  <summary>Step 4</summary>
  
  Evaluate the code.
</details>