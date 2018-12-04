TypeScript新手入門
===

###### tags: `typeScript` `javascript`

<style>

[class='lang-javascript= hljs']{
	font-size:24px !important;
}

[class='lang-javascript=middle hljs']{
	font-size:20px !important;
}

[class='lang-javascript=small hljs']{
	font-size:16px !important;
}

</style>

---

## 為什麼要使用TypeScript?

----

在程式語言之中，有區分為強型別和弱型別兩種類型。

----

JavaScript是弱型別語言

* 優點是寫法超彈性

* 缺點則是無法在開發時期檢查型別

----


```javascript=
	var x = 1;
	var y = '2';
	
	function add(a,b){
		return a + b;
	}
	
	console.log(add(x,y)); //結果變成用字串相加，印出12
	
```

會在【執行時期】自己判斷目前應該執行的型別

<!-- .element: class="fragment" data-fragment-index="1" -->

自行做【型別自動轉換】

<!-- .element: class="fragment" data-fragment-index="1" -->

但在快速開發時，很有可能引發一些預期之外的錯誤。

<!-- .element: class="fragment" data-fragment-index="2" -->

相當不利於多人協同開發。

<!-- .element: class="fragment" data-fragment-index="2" -->

----

強型別語言比較著名的有像是 C#、Java 等等，

它們對於變數的定義檢查比較嚴謹，

如果沒有依照指定的型別傳入參數，

在【編譯時期】就會拋出 Exception 警告開發人員。

![](https://i.imgur.com/tMzP0Kw.png)

<!-- .element: class="fragment" data-fragment-index="1" -->

----

**TypeScript是一個提供強型別語法的JavaScript超集合**

![](https://i.imgur.com/tKcLkTY.png)

----

### TypeScript優點

* 在編譯時期就能做型別檢查，提升程式碼品質
<!-- .element: class="fragment" data-fragment-index="1" -->

* 透過【預編譯】器，向下相容舊有的JavaScript寫法
<!-- .element: class="fragment" data-fragment-index="2" -->

* 直接與ES6完美結合，並提供更多支援寫法
<!-- .element: class="fragment" data-fragment-index="3" -->

* 二大IT龍頭支援：Microsoft及Google
<!-- .element: class="fragment" data-fragment-index="4" -->

---

## 基本型別

----

支援所有JS型別(boolean、number、string、Array)

還提供了enum(列舉)、any(任意型別)、void(無型別)

<!-- .element: class="fragment" data-fragment-index="1" -->

所有的變數都可以在宣告的時候明確【指定變數的型別】

<!-- .element: class="fragment" data-fragment-index="2" -->

如果指派了不符合的型別內容，編譯器就會發出錯誤提示

<!-- .element: class="fragment" data-fragment-index="3" -->

![](https://i.imgur.com/HMBP089.png)

<!-- .element: class="fragment" data-fragment-index="3" -->

----

### boolean

```javascript=
var isDone:boolean = false;
```

----

### number

與JS一樣，在TS中所有的數值都是【浮點數】

```javascript=
var min:number = 60;

var pi:number = 3.14;

```

----

### string

與JS一樣，可以使用雙引號(")或單引號(')表示字串

```javascript=
var name:string = "lala";
name = 'momo';
```

----

跟ES6一樣，可以使用【字串樣版 Template Literals】

它可以定義多行文字和內嵌運算式

<!-- .element: class="fragment" data-fragment-index="1" -->

* 使用｀(重音符號) 包圍文字，表示為字串樣版。

* 在字串樣版中可以任意使用 ${variable} 放置變數。

<!-- .element: class="fragment" data-fragment-index="2" -->

```javascript=
var name: string = `Gene`;
var age: number = 37;
var sentence: string = `Hello, my name is ${ name }.
I'll be ${ age + 1 } years old next month`.;
```

<!-- .element: class="fragment" data-fragment-index="3" -->

----

### Array

有二種方式可以定義陣列

1. 在想宣告的型別後面加上[]，宣告為某個型別的 Array

	```javascript=
	// 宣告一個 number 的 Array
	var idList: number[] = [1, 2, 3];
	```

2. 使用陣列泛型，Array<元素型別>

	```javascript=
	var list: Array<number> = [1, 2, 3];
	```

----

以下範例的結果為何？

```javascript=
var list: Array<number> = [1, 2, 3];
list.push("4");
```

<!-- .element: class="fragment" data-fragment-index="1" -->

![](https://i.imgur.com/yNToQ8o.png)

<!-- .element: class="fragment" data-fragment-index="2" -->

----

### enum 列舉型別

enum是對JavaScript標準資料型別的一個擴充。

使用enum可以為一組數值賦予有意義的名稱。

```javascript=
enum Color {Red, Green, Blue};
var c: Color = Color.Green;
```

----

預設情況下，從0開始為元素編號。

```javascript=
enum Color {Red , Green, Blue};
//Color.Red = 0,Color.Green = 1 ,Color.Blue = 2
```

你也可以手動指定成員的數值。(手動指定的成員只會干涉之後的成員)

<!-- .element: class="fragment" data-fragment-index="1" -->

```javascript=
enum Color {Red = 1, Green, Blue};
//Color.Red = 1,Color.Green = 2 ,Color.Blue = 3
enum Color {Red, Green = 2, Blue};
//Color.Red = 0,Color.Green = 2 ,Color.Blue = 3
```

<!-- .element: class="fragment" data-fragment-index="1" -->

或者，全部都採用手動賦值

<!-- .element: class="fragment" data-fragment-index="2" -->

```javascript=
enum Color {Red = 1, Green = 2, Blue = 4};
//Color.Red = 1,Color.Green = 2 ,Color.Blue = 4
```

<!-- .element: class="fragment" data-fragment-index="2" -->

----

### any 任意型別

有些來自第三方提供的函式庫，

我們不希望型別檢查器對這些值進行檢查，

那麼我們可以用 any 來標記這些變數。

<!-- .element: class="fragment" data-fragment-index="1" -->

```javascript=
var notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // 因為型別指定為any所以不會報錯。
```

<!-- .element: class="fragment" data-fragment-index="1" -->

只建議在快速開發時使用，為了程式的穩定還是少用。

<!-- .element: class="fragment" data-fragment-index="2" -->

----

### void

通常用在當函式沒有回傳值時。

```javascript=
function showalert:void {
	alert("this is the message");
}
```

---

## 基本語法

----

### 變數 

* var (function scope)
* let (block scope)
* const (常數)

----

### let

**使用let取代var**

----

#### let不會有hoisting的問題 ####

在 javascript 中 hoisting(提升)指的是

【變數宣告會被隱含提升到其所在區域內的頂端】

原因：變數和函數的宣告會在編譯階段就被放入記憶體

[JavaScript Hoisting - W3Schools](http://www.w3schools.com/js/js_hoisting.asp)

----

![](https://i.imgur.com/iCWKeoi.png)

在執行時期時，所有var變數都會自動被hoisting。

僅提升宣告的部分，而不會執行初始化動作。

因此若程式中有參考到使用var定義過的變數時，

會變成undefined，不會產生ERROR，

容易在DEBUG時造成誤解。

----

![](https://i.imgur.com/2LKe4hX.png)

使用let不會被hoisting，它是依附在{}的區塊中，

若程式中有參考到let定義過的變數時，因作用區塊不同

會產生ERROR，此行為比較接近常用的程式語言寫法。

----

#### let在相同scope時，不會被重新宣告 ###

----

**在for loop中使用var**

迴圈先跑完，var的值會redeclared，

所以callback function裡的i，永遠會是最後一個值。

```javascript=

var names = ['a','b','c'];

function asyncFn(names){
	//...
	for (var i in names){
		$.get("/users/" + names [i],function(){
			console.log("ajax get for",names[i]);
		});
	}
	// result is as follow
	// ajax get for c
	// ajax get for c
	// ajax get for c
}
```

----

**在for loop中改用let**

迴圈先跑完，let的值不會redeclared，

所以callback function裡的i，會依附著for loop做變化。

```javascript=

let names = ['a','b','c']; //這裡用不用let都沒有影響。

function asyncFn(names){
	//...
	for (let i in names){
		$.get("/users/" + names [i],function(){
			console.log("ajax get for",names[i]);
		});
	}
	// result is as follow
	// ajax get for a
	// ajax get for b
	// ajax get for c
}
```

----

請問下列程式碼結果為何

```javascript=
	let msg = "web forum";
	function printInCaps(value){
		let msg = value.toUpperCase();
		return msg;
	}
	
	printInCaps("profiles");
	console.log(msg);
```

----

### const

使用const定義常數值，更容易理解。

----

不好的寫法

![](https://i.imgur.com/gyCWn6p.png)

----

改用const會讓程式可容易閱讀。

![](https://i.imgur.com/zGrPjUD.png)

----

const是唯讀的，一旦定義了就無法改變。

![](https://i.imgur.com/jgSmRHB.png)

----

宣告const的時候，一定要指定初值，

否則會產生ERROR。

![](https://i.imgur.com/M2WMpXK.png)

----

const也不會有hoisting的問題。

![](https://i.imgur.com/cer8WHo.png)

----

### Function 

----

#### 為函式定義型別

與原本 JavaScript Function 不同的是，

TypeScript需要指定【傳入參數】和【回傳值】的型別，

避免手誤傳入不正確的參數而導致異常。

![](https://i.imgur.com/yjkJgaV.png)

----

#### 選擇性參數和預設參數

----

```javascript=
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

var result3 = buildName("Bob", "Adams");  
//correct
var result1 = buildName("Bob");  
//error, too few
var result2 = buildName("Bob", "Adams", "Sr.");  
//error, too many 

```

在TypeScript中，「函式呼叫時傳遞的參數個數

<!-- .element: class="fragment" data-fragment-index="1" -->

必須與函式期望的參數數量一致」，否則編譯器會報錯。

<!-- .element: class="fragment" data-fragment-index="1" -->

----

JavaScript裡，每個參數都是選擇性的。

沒傳值的時候，它的值就是undefined。

在TypeScript要實現【選擇性參數】的功能，

必須要在參數名稱使用"?"，標示此參數為optional

```javascript=
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

var result1 = buildName("Bob", "Adams");  
//correct, "Bob Adams"
var result2 = buildName("Bob");  
//correct, "Bob"
```

----

或者，我們也可以在參數後面【指定預設值】

當傳入參數沒指定時，可以根據預設值來做處理

```javascript=
function buildName(firstName: string, lastName: string = '') {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

var result1 = buildName("Bob", "Adams");  
//correct, "Bob Adams"
var result2 = buildName("Bob");  
//correct, "Bob"
```

----

特別需要注意的是，

* 選擇性參數 `lastname?:string`

* 有預設值的參數 `lastname:string=''`

都必需要【放在Function的尾端】，否則會報錯。

----

#### 其餘參數 (rest parameter)

----

ES6提供了**其餘參數(Rest parameters)** 

是一種參數的語法：「不確定的傳入參數值的個數」

此參數會把傳給它的"其餘"的所有值，

轉換成一個【數值陣列】。

```javascript=
function sum(...numbers) {
  let result = 0;
  numbers.forEach(function (number) {
    result += number;
  });
  return result;
}
console.log(sum(1)); // 1
console.log(sum(1, 2, 3, 4, 5)); // 15
```

----

如同ES6一樣，TS也可以在函式中使用其餘參數

標示為rest param的參數，型別必須要是Array

<!-- .element: class="fragment" data-fragment-index="1" -->

```javascript=
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

var employeeName = buildName("Joseph", "Samuel", "Lucas");
```

<!-- .element: class="fragment" data-fragment-index="1" -->

----

#### arrow function

arrow function ()=>

寫法如下

```javascript=

fnName(x,y) => {
	//...
}

```
----

* 傳統寫法與ES6寫法的比較

```javascript=
//ES5
function foo(x, y) { 
    x++;
    y--;
    return x + y;
}

//ES6
foo(x, y) => {
	x++; 
	y--; 
	return x+y;
}
```

----

#### arrow function解決了this指向的問題

此範例中的this，

因為被settimeout callback function影響的關係，

this變成指向window.this。

```javascript=
//沒有使用arrow function

class Animal {
    constructor(){
        this.type = 'animal'
    }
    says(say){
        setTimeout(function(){
            console.log(this.type + ' says ' + say)
        }, 1000)
    }
}

 var animal = new Animal()
 animal.says('hi')  //undefined says hi

```

----

用了()=>之後，函數體內的this，

【就是定義時所在的對象】，而不是使用時所在的對象。

()=>不會有自己的this，它的this是繼承外面的，

因此內部的this就是外層代碼區域的this。

```javascript=
class Animal {
    constructor(){
        this.type = 'animal'
    }
    says(say){
        setTimeout( () => {
            console.log(this.type + ' says ' + say)
        }, 1000)
    }
}
 var animal = new Animal()
 animal.says('hi')  //animal says hi
```

----

#### for...of and for...in

**for...of**是遍歷**屬性值(property values)**

**for...in**是遍歷**屬性名(property names)**

![](https://i.imgur.com/sLYg5CM.png)

---

## class

----

### 基本語法

寫法同ES6，會有屬性、建構式的特性

```javascript=
class Student {
  name: string;

  constructor(name: string){
    this.name = name;
  }

  getPhoneNumber(): string {
    //程式邏輯
  }
}
```

----

### property 屬性

屬性預設都是public，不要被外部存取，要加上private。

```javascript=
class Student {  
  name: string;    
  age: number;
  private  birth: Date;  
}

var student = new Student();

console.log(student.name);  //正常
console.log(student.birth); //錯誤提示
```

----

也可以把屬性宣告為static，

在使用時，也不需要實體化就能夠呼叫。

```javascript=
class Student {  
  static type: string;
}

Student.type;
```

----

想要給予屬性預設值的話，可以使用等號直接指定。

```javascript=
class Student {  
  name: string = 'Student';
}
```

----

### constructor 建構式

```javascript=
class Student {

	name: string;  
	age: number;

	constructor(name: string, age: number) {    
    	this.name = name;    
    	this.age = age;  
	}
}

var student = new Student('Kirk', 18);
console.log(`Student name: ${student.name}, age: ${student.age}`);
```

----

以上寫法，TypeScript提供了速寫語法。

```javascript=
class Student {  
  constructor(public name: string,public age: number) {}
}

var student = new Student('Kirk', 18);
console.log(`Student name: ${student.name}, age: ${student.age}`);

```

---

## interface 

----

### interface basic

同Java的寫法相似，interface(介面)是一種規格

```javascript=
interface IClock {
    currentTime: Date;
    setTime(d: Date);
}

```

----

當一個類別實踐一個介面，表示他必須實踐這個規格。

實做類別的關鍵字是 implements

```javascript=
class Clock implements IClock  {
    currentTime: Date;
    constructor(
		public h: number, 
		public m: number) {
	}
	setTime(d: Date) {
        this.currentTime = d;
    }
}

var myClock: IClock = new Clock(12,60);
myClock.setTime(new Date());
```

----

### hybrid types

另一種用法：利用介面來達成複雜型別(complex type)

```javascript=
interface IPerson{
	name:string;
	age:number;
}

function showPerson(person:IPerson){
	console.log("name: ",person.name," age:",person.age);
}
```

----

### 介面擴充

就像類別一樣，介面也可以彼此擴充。

它可以讓你快速的複製一個介面的成員到另一個介面

而且這樣更利於介面的重用性，讓程式更有彈性。

```javascript=
interface Shape {
	color:string;
}

interface Square extends Shape {
	sideLength:number;
}

// 這個例子是把interface當做複雜型別
// <型別> 是泛型的用法，後面會再提到
let square = <Square>{};
square.color = 'blue';
square.sideLength = 10;
```

---

## Generic

----

當你在建構Function、Interface及Class時

你會希望這些Component都是能夠被重複運用的

Generic(泛型)提供了一個彈性的做法

語法是：`<T>`

----

你在撰寫Function、Interface及Class時

不指定變數的【具體型別】，

而是用一個【型別參數】做代替 

未來在使用這個Component時，

這個【型別參數】會由一個具體的型別所代替。

----

### Generic Functions

```javascript=
function LogAndReturn<T>(thing:T):T {
	console.log(thing);
	return thing;
}

let stringLog:string = LogAndReturn<string>('string log');

let numberLog:number = LogAndReturn<number>(1000);

let newBook:Book = {title:'This is a Book'};
let bookLog:Book = LogAndReturn<Book>(newBook);
```

----

### Generic Interfaces

```javascript=
interface Inventory<T> {
	addItem:(newItem:T) => void;
	getAllItems:() => Array<T>;
}

let bookInventory:Inventory<Book>;
let allBooks:Array<Book> = bookInventory.getAllItems();
let comicInventory:Inventory<Comic>;
let allComics:Array<Comic> = comicInventory.getAllItems();
```

----

### Generic Classes

```javascript=middle
class Shelf<T> {
	private _items:Array<T> = new Array<T>();
	
	add(item:T):void{
		this._items.push(item);
	}
}

let newBook:Book = {title:'CSS Secrets'};
let bookShelf:Shelf<Book> = new Shelf<Book>();
bookShelf.add(newBook);

let newNumber:number = 1000;
let numberShelf:Shelf<number> = new Shelf<number>();
numberShelf.add(newNumber);
```

----

有時候你的商業邏輯需要操作宣告為泛型的類別成員。

```javascript=
class Shelf<T> {
	private _items:Array<T> = new Array<T>();
	
	add(item:T):void{this._items.push(item);}
	
	printTitles():void{
		this._items.forEach(
			item => console.log(item.title)
		);
		//compile會告訴你T沒有title這個屬性
	}
}

```

遇到這種狀況，你要做的是為Generic加上限制。

<!-- .element: class="fragment" data-fragment-index="1" -->

----

### Generic Constraints

```javascript=middle

interface ShelfItem {
	title:string;
}

class Shelf<T extends ShelfItem> {
	private _items:Array<T> = new Array<T>();
	
	add(item:T):void{this._items.push(item);}
	
	printTitles():void{
		this._items.forEach(
			item => console.log(item.title)
		);
	}
}

```

----

```javascript=middle

let newBook:Book = {title:'CSS Secrets'};
let bookShelf:Shelf<Book> = new Shelf<Book>();
bookShelf.add(newBook);
bookShelf.printTitles();//Correct，因為Book有title的屬性

let newNumber:number = 1000;
let numberShelf:Shelf<number> = new Shelf<number>();
numberShelf.add(newNumber);
numberShelf.printTitles();//Error，因為number沒有title的屬性
```

---

## Modules

----

在TS中【組織程式碼的方法】

* 外部模組 - module 

	模組之間是不同功能，利用import/export來互相引用彼此公開的功能

* 內部模組 - namespace

	模組之間是相近的功能，使用namespace集中功能。

----

### moudle 外部模組

* 使用 export (用法同ES6)
	
	將想要分享的變數、函式、類別及介面做 **公開**

* 使用 import (用法同ES6)

	**引用** 不同檔案並設定公開的變數、函式、類別及介面
	
*MyExport.ts*

<!-- .element: class="fragment" data-fragment-index="1" -->

```javascript=
	export class SomeType  { /* ... */ }
	export function someFn { /* ... */ }
```

<!-- .element: class="fragment" data-fragment-index="1" -->

*App.ts*

<!-- .element: class="fragment" data-fragment-index="2" -->

```javascript=
	import { SomeType,somefn } from './Myexport';
	let x = new SomeType();
	let y = someFn();
```

<!-- .element: class="fragment" data-fragment-index="2" -->

----

### namespace 內部模組(命名空間)

----

請先觀察以下寫法，有什麼缺點。

```javascript=small

interface Shape {
	area(h:number,w:number):number;
}

class Square implements Shape {
	area(h:number,w:number) {return h*w;}
}

class Triangle implements Shape { 
	area(h:number,w:number) {return (h*w) / 2;}
}

let s = new Square();
console.log(s.area(10,5)); // 50
let t = new Triangle();
console.log(t.area(10,5)); // 25
```

Shape、Square、Triangle 放在 global namespace

<!-- .element: class="fragment" data-fragment-index="1" -->

放在global namespace的缺點是容易造成名稱衝突

<!-- .element: class="fragment" data-fragment-index="2" -->

----

前例的寫法可修改成模組化，關鍵字使用namespace

```javascript=small
namespace Geometric { 
	const HALF = 0.5;
	
	export interface Shape {
		area(h:number,w:number):number;
	} 
	
	export class Square implements Shape {
		area(h:number,w:number) {return h*w;}
	}

	export class Triangle implements Shape { 
		area(h:number,w:number) { return (h*w)*HALF };
	}
} //所以global namespace，只有Geometric這個物件

let s = new Geometric.Square();
console.log(s.area(10,5)); // 50
let t = new Geometric.Triangle();
console.log(t.area(10,5)); // 25
```

我們希望介面跟類別是公開的，所以使用export公開。

<!-- .element: class="fragment" data-fragment-index="1" -->

而變數HALF是實現的細節，就不必要使用export，

<!-- .element: class="fragment" data-fragment-index="2" -->

因此變數HALF在模組外是不可見的。

<!-- .element: class="fragment" data-fragment-index="2" -->


----

![](https://i.imgur.com/M1IlGQI.png)

可以觀察到編譯成ES3之後，

模組是被包裝成立即函式，因此避免了全域環境汙染。

----

將模組拆分成多個檔案

隨著應用的擴展，我們希望將程式拆分成多個文件

使每個檔案的功能更單純，更方便維護。(單一職責原則)

----

*Shape.ts*
```javascript=small
namespace Geometric {
	export interface Shape {
		area(h:number,w:number):number;
	} 
}
```

*Square.ts*
```javascript=small
/// <reference path="Shape.ts" />
namespace Geometric {
	export class Square implements Shape {
		area(h:number,w:number) {return h*w;}
	}
}
```

*Triangle.ts*
```javascript=small
/// <reference path="Shape.ts" />
namespace Geometric {
	export class Triangle implements Shape {
		area(h:number,w:number) {return (h*w)/2;}
	}
}
```

----

雖然被拆分成多個檔案，每個檔案都是單獨的

但是他們都在為同一個模組貢獻功能。

檔案中有參考的地方，使用 reference 來做引用

----

*App.ts*
```javascript=small
/// <reference path="Shape.ts" />
/// <reference path="Square.ts" />
/// <reference path="Triangle.ts" />

let s = new Geometric.Square();
console.log(s.area(10,5)); // 50
let t = new Geometric.Triangle();
console.log(t.area(10,5)); // 25
```

----
將模組拆分成多個檔案後使用 `tsc App.ts` 語法編譯後執行會出現錯誤:
```
var s = new Geometric.Square();
        ^

ReferenceError: Geometric is not defined
```
解決方法是參考 [Namespaces · TypeScript](https://www.typescriptlang.org/docs/handbook/namespaces.html) 使用 `tsc --outFile App.js App.ts` 
____

### 別名

當取用模組的path比較長時，

可以使用 `import q = x.y.z` 的語法

給常用的模組起一個簡短的名稱

```javascript=middle
	namespace Shapes {
		export namespace Polygons {
			export class Square {}
			export class Triangle {}
		}
	}
	
	//沒有用別名之前
	var test1 = new Shapes.Polygons.Square();
	var test2 = new Shapes.Polygons.Triangle();
	
	//使用別名之後
	import pg = Shapes.Polygons;
	var sq = new pg.Square();
	var tri = new pg.Triangle();
```

---

## Type Script Compiler (tsc)

* 瀏覧器不會直接執行TypeScript，tsc會將TypeScript轉譯成JavaScript

* 所有轉譯的設定，被定義在tsconfig.json這個檔案中

----

### tsconfig.json

基本設定

```javascript=middle
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "typeRoots": [
      "../../node_modules/@types/"
    ]
  },
  "compileOnSave": true,
  "exclude": [
    "node_modules/*"
  ]
}

```

---

## Type Definition File

* 當你操作第三方函式庫(Lib)時，都需要定義檔(*.d.ts)

* 定義檔用來揭露第三方函式庫的型別資訊

* 不包含實作的細節，可以想像它是c/c++的表頭檔(*.h)

* 有了Lib定義檔，就可在開發時期操作時Lib做型別檢查


----

### Ambient Declaration 

ambient declaration = 周圍環境宣告

代表你要宣告一個在目前環境中已經有的變數，

例如jQuery你早就載入了，只是TS不知道，

透過這招設定讓TS知道$代表某個型別

*jquery.d.ts*
```javascript=
declare var jQuery: JQueryStatic;
declare var $: JQueryStatic;
```

*knockout.d.ts*
```javascript=
declare var ko: KnockoutStatic;
```

----

### Type Definition Sources

![](https://i.imgur.com/OgCy7z4.png)

----

#### [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped)

* Github儲存庫，存放很多定義檔，可手動下載至專案內
	
* 舊的作法，目前不建議使用此方式下載定義檔

----

#### [tsd](https://www.npmjs.com/package/tsd)

* NPM套件，可用來搜尋及下載TS的定義檔

* 所有下載的設定，可以在tsd.json看到

* 舊的作法，目前不建議使用此方式下載定義檔


----

#### [typings](https://www.npmjs.com/package/typings)

* NPM套件，可用來搜尋及下載TS的定義檔

* 可以指定多個下載來源

	* npm/github/bower/dt(DefinitelyTyped)

* 所有下載的設定，可以在typings.json看到

* 舊的作法，目前不建議使用此方式下載定義檔

----

#### [@types](https://www.npmjs.com/~types)

* 在Typescript 2.0之後，盡量不使用tsd及typings

* 不需再額外安裝其他下載定義檔的套件，直接使用npm

	```npm install --save @types/lodash```

* 所有下載的設定，可以直接在package.json看到

* 只需要在tsconfig.json指定typeRoots，tsc就知道去哪裡找定義檔

```
    "typeRoots": [
      "../node_modules/@types"
    ]
```

---

## 小結

* JavaScript慢慢變成類似Assembly Language的地位

* 我們會用ES2015或是TypeScript的高階語法來開發JS

* 透過【預編譯】把高階語法轉譯成瀏覧器認識的ES3or5

* TS的型別檢查、模組化及OOP特性，增加程式開發品質。 


