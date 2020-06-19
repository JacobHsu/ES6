# let

ES6 新增了`let`命令，用來聲明變量。它的用法類似於`var`，但是所聲明的變量，只在let命令所在的代**碼塊內**有效。

```js
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

上面代碼在代碼塊之中，分別用let和var聲明了兩個變量。然後在代碼塊之外調用這兩個變量，結果let聲明的變量報錯，var聲明的變量返回了正確的值。這表明，let聲明的變量只在它所在的代碼塊有效。

## 基本用法

`for`循環的計數器，就很合適使用`let`命令

```js
for (let i = 0; i < 10; i++) {
  // ...
}

console.log(i);
// ReferenceError: i is not defined
```
上面代碼中，計數器`i`只在`for`循環體內有效，在循環體外引用就會報錯。

下面的代碼如果使用var，最後輸出的是`10`。

```js
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```

上面代碼中，變量i是var命令聲明的，在全局範圍內都有效，所以全局只有一個變量i。每一次循環，變量i的值都會發生改變，而循環內被賦給數組a的函數內部的console.log(i)，裡面的i指向的就是全局的i。也就是說，所有數組a的成員裡面的i，指向的都是同一個i，導致運行時輸出的是最後一輪的i的值，也就是 `10`。

如果使用let，聲明的變量僅在塊級作用域內有效，最後輸出的是 `6`。

```js
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```

上面代碼中，變量i是let聲明的，當前的i只在本輪循環有效，所以每一次循環的i其實都是一個新的變量，所以最後輸出的是`6`。

你可能會問，如果每一輪循環的變量i都是重新聲明的，那它怎麼知道上一輪循環的值，從而計算出本輪循環的值？這是因為 JavaScript 引擎內部會記住上一輪循環的值，初始化本輪的變量i時，就在上一輪循環的基礎上進行計算。

> for循環還有一個特別之處，就是設置循環變量的那部分是一個父作用域，而循環體內部是一個單獨的子作用域。

```js
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```

上面代碼正確運行，輸出了 3 次abc。這表明函數內部的變量i與循環變量i不在同一個作用域，有各自單獨的作用域。

## 不存在变量提升

`var`命令會發生“變量提升”現象，即變量可以在聲明之前使用，值為undefined。這種現象多多少少是有些奇怪的，按照一般的邏輯，變量應該在聲明語句之後才可以使用。

為了糾正這種現象，`let`命令改變了語法行為，它所聲明的變量一定要在聲明後使用，否則報錯。

```js
// var 的情況
console.log(foo); // 輸出undefined
var foo = 2;

// let 的情況
console.log(bar); // 報錯ReferenceError
let bar = 2;
```

上面代碼中，變量foo用`var`命令聲明，會發生變量提升，即腳本開始運行時，變量foo已經存在了，但是沒有值，所以會輸出`undefined`。
變量bar用`let`命令聲明，不會發生變量提升。這表示在聲明它之前，變量bar是不存在的，這時如果用到它，就會拋出一個錯誤`ReferenceError`。

## 暂时性死区 (TDZ)

只要塊級作用域內存在let命令，它所聲明的變量就`“綁定”（binding）`這個區域，不再受外部的影響。

```js
var tmp = 123;

if (true) {
  tmp = 'abc'; // ReferenceError
  let tmp;
}
```

上面代碼中，存在全局變量tmp，但是塊級作用域內let又聲明了一個局部變量tmp，導致後者綁定這個塊級作用域，所以在let聲明變量前，對tmp賦值會報錯。

ES6 明確規定，如果區塊中存在`let`和`const`命令，這個區塊對這些命令聲明的變量，從一開始就形成了封閉作用域。凡是在聲明之前就使用這些變量，就會報錯。

總之，在代碼塊內，使用let命令聲明變量之前，該變量都是不可用的。這在語法上，稱為“暫時性死區”（`temporal dead zone`，簡稱 `TDZ`）。

```js
if (true) {
  // TDZ開始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ結束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

上面代碼中，在let命令聲明變量tmp之前，都屬於變量tmp的“死區”。

### typeof

**暫時性死區**也意味著typeof不再是一個百分之百安全的操作。

```js
typeof x; // ReferenceError
let x;
```

上面代碼中，變量x使用`let`命令聲明，所以在聲明之前，都屬於x的“死區”，只要用到該變量就會報錯。因此，`typeof`運行時就會拋出一個`ReferenceError`。

作為比較，如果一個變量根本沒有被聲明，使用`typeof`反而不會報錯。

```js
typeof undeclared_variable // "undefined"
```

上面代碼中，undeclared_variable是一個不存在的變量名，結果返回“undefined”。所以，在沒有let之前，`typeof`運算符是百分之百安全的，永遠不會報錯。現在這一點不成立了。

這樣的設計是為了讓大家養成良好的編程習慣，**變量一定要在聲明之後使用，否則就報錯。**

### 隱蔽型死區

```js
function bar(x = y, y = 2) {
  return [x, y];
}

bar(); // 報錯 "ReferenceError: Cannot access 'y' before initialization
```

上面代碼中，調用bar函數之所以報錯（某些實現可能不報錯），是因為參數x默認值等於另一個參數y，而此時y還沒有聲明，屬於“死區”。如果y的默認值是x，就不會報錯，因為此時x已經聲明了

```js
function bar(x = 2, y = x) {
  return [x, y];
}
bar(); // [2, 2]
```

另外，下面的代碼也會報錯，與`var`的行為不同。

```js
// 不報錯
var x = x;

// 報錯
let x = x;
// ReferenceError: x is not defined
```

上面代碼報錯，也是因為暫時性死區。使用let聲明變量時，只要變量在還沒有聲明完成前使用，就會報錯。上面這行就屬於這個情況，在變量x的聲明語句還沒有執行完成前，就去取x的值，導致報錯”x 未定義“。

ES6 規定暫時性死區和let、const語句不出現變量提升，主要是為了減少運行時錯誤，防止在變量聲明前就使用這個變量，從而導致意料之外的行為。這樣的錯誤在 ES5 是很常見的，現在有了這種規定，避免此類錯誤就很容易了。

總之，暫時性死區的本質就是，只要`一進入當前作用域，所要使用的變量就已經存在了，但是不可獲取`，只有等到聲明變量的那一行代碼出現，才可以獲取和使用該變量。
