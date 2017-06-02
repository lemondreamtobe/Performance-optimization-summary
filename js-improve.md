## HTML 和 DOM 优化
### HTML 文件中插入 JAVASCRIPT 代码

- JAVASCRIPT 脚本应与HTML文件相分离。
- 尽量 JAVASCRIPT 脚本放在底部，以防止 JAVASCRIPT 阻塞。
- JAVASCRIPT 中使用`document.write`生成页面内容会效率较低，可以找一个容器元素，比如指定一个`div`，并使用`innerHTML`来将`HTML`代码插入到页面中。
- 多使用`innerHTML`,虽然这并不是`W3C`的标准。
	- 有两种在页面上创建`DOM`节点的方法：
		- 使用诸如`createElement()`和`appendChild()`之类的`DOM`方法。
		- 使用`innerHTML`。
			- 当使用`innerHTML`设置为某个值时，后台会创建一个`HTML`解释器，然后使用内部的`DOM`调用来创建`DOM`结构，而非基于`JAVASCRIPT`的`DOM`调用。由于内部方法是编译好的而非解释执行，故执行的更快。
		>对于小的`DOM`更改，两者效率差不多，但对于大的`DOM`更改，`innerHTML`要比标准的`DOM`方法创建同样的`DOM`结构快得多。

#### 优化节点修改：

使用`cloneNode`在外部更新节点然后再通过`replace`与原始节点互换，避免反复`remove`和`insert`。

		var orig = document.getElementById('container');
		var clone = orig.cloneNode(true);
		var list = ['foo', 'bar', 'baz'];
		var content;
		for (var i = 0; i < list.length; i++) {
		   content = document.createTextNode(list[i]);
		   clone.appendChild(content);
		}
		orig.parentNode.replaceChild(clone, orig);

#### 优化节点添加：

多个节点插入操作，即使在外面设置节点的元素和风格再插入，由于多个节点还是会引发多次reflow。优化的方法是创建`DocumentFragment`，在其中插入节点后再添加到页面。这里以 JQuery 中所有的添加节点的操作如`append` 为例，都是最终调用`DocumentFragment`来实现的，

		
		var safeFrag = document.createDocumentFragment();
		for (var i = 0; i < 3; i++) {
			var a = document.createElement("div");
			a.id = i;
			safeFrag.appendChild(a);
		};
		document.body.appendChild(safeFrag);

#### 优化`CSS`样式转换。

如果需要动态更改CSS样式，尽量采用触发reflow次数较少的方式，如以下代码逐条更改元素的几何属性，理论上会触发多次`reflow`。

		element.style.fontWeight = 'bold' ;
		element.style.marginLeft= '30px' ;
		element.style.marginRight = '30px' ;

可以通过直接设置元素的`className`直接设置，只会触发一次`reflow`。

		element.className = 'selectedAnchor' ;

#### 减少`DOM`元素数量：

在`console`中执行命令查看`DOM`元素数量。

		document.getElementsByTagName( '*' ).length

在一个正常页面的`DOM`元素数量一般不应该超过`1000`，`DOM`元素过多会使`DOM`元素查询效率，样式表匹配效率降低，是页面性能最主要的瓶颈之一。

#### DOM 操作优化。

- 最小化`DOM`访问次数，尽可能在js端执行。
- 如果需要多次访问某个`DOM`节点，请使用局部变量存储对它的引用。
- 谨慎处理`HTML`集合（`HTML`集合实时连系底层文档），把集合的长度缓存到一个变量中，并在迭代中使用它，如果需要经常操作集合，建议把它拷贝到一个数组中。		     
- 如果可能的话，使用速度更快的API，比如`querySelectorAll`和`firstElementChild`。
- 使用事件委托来减少事件处理器的数量。
- 将获取的`DOM`数据缓存起来。这种方法，对获取那些会触发回流操作的属性（比如`offsetWidth`等）尤为重要。  
- 当对HTMLCollection对象进行操作时，应该将访问的次数尽可能的降至最低，最简单的，你可以将length属性缓存在一个本地变量中，这样就能大幅度的提高循环的效率。

###事件优化
通过冒泡机制，在上层文档对底层事件统一管理，避免对每个子节点都注册事件。
		<ul id="parent-list">
			<li id="post-1">Item 1</li>
			<li id="post-2">Item 2</li>
			<li id="post-3">Item 3</li>
			<li id="post-4">Item 4</li>
			<li id="post-5">Item 5</li>
			<li id="post-6">Item 6</li>
		</ul>
		// Get the element, add a click listener...
		document.getElementById("parent-list").addEventListener("click",function(e) {
			// e.target is the clicked element!
			// If it was a list item
			if(e.target && e.target.nodeName == "LI") {
				// List item found!  Output the ID!
				console.log("List item ",e.target.id.replace("post-")," was clicked!");
			}
		});

###内存优化

- `JAVASCRIPT`的内存泄露处理
	- 给`DOM`对象添加的属性是一个对象的引用。
>
			var MyObject = {};
			document.getElementByIdx_x('myDiv').myProp = MyObject;
>
		解决方法：在window.onunload事件中写上: 
>
			document.getElementByIdx_x('myDiv').myProp = null;
>
	- DOM对象与JS对象相互引用。
>
			function Encapsulator(element) {
			   this.elementReference = element;
			   element.myProp = this;
			}
			new Encapsulator(document.getElementByIdx_x('myDiv'));
>
		解决方法：在onunload事件中写上: 
>
			document.getElementByIdx_x('myDiv').myProp = null;
>
	- 给DOM对象用attachEvent绑定事件。
>
			function doClick() {}
			element.attachEvent("onclick", doClick);
>
		解决方法：在onunload事件中写上: 
>
			element.detachEvent('onclick', doClick);
>
	- 从外到内执行appendChild。这时即使调用removeChild也无法释放。
>
			var parentDiv =   document.createElement_x("div");
			var childDiv = document.createElement_x("div");
			document.body.appendChild(parentDiv);
			parentDiv.appendChild(childDiv);
>
		解决方法：从内到外执行appendChild:
>
			var parentDiv =   document.createElement_x("div");
			var childDiv = document.createElement_x("div");
			parentDiv.appendChild(childDiv);
			document.body.appendChild(parentDiv);
>	 
	- 反复重写同一个属性会造成内存大量占用(但关闭IE后内存会被释放)。
>
			for(i = 0; i < 5000; i++) {
			   hostElement.text = "asdfasdfasdf";
			}
>
		这种方式相当于定义了5000个属性，解决方法：无。
- `内存`不是`缓存`。
	- 不要轻易将`内存`当作`缓存`使用。
	- 如果是很重要的资源，请不要直接放在`内存`中，或者制定`过期机制`，自动销毁`过期缓存`。
- `CollectGarbage`。
	- `CollectGarbage`是`IE`的一个特有属性,用于释放内存的使用方法,将该变量或引用对象设置为`null`或`delete`然后在进行释放动作，在做`CollectGarbage`前,要必需清楚的两个必备条件:（引用）。
		- 一个对象在其生存的上下文环境之外，即会失效。
		- 一个全局的对象在没有被执用(引用)的情况下，即会失效
		
###作用域优化
在实际项目中我们应该减少作用域链上的查找次数，JAVASCRIPT 代码在执行的时候，如果需要访问一个变量或者一个函数的时候，它需要遍历当前执行环境的作用域链，而遍历是从这个作用域链的前端一级一级的向后遍历，直到全局执行环境。

	/**效率低**/
	for(var i = 0; i < 10000; i++){
	    var but1 = document.getElementById("but1");
	}
	/**效率高**/
	/**避免全局查找**/
	var doc = document;
	for(var i = 0; i < 10000; i++){
	    var but1 = doc.getElementById("but1");
	}

上面代码中，第二种情况是先把全局对象的变量放到函数里面先保存下来，然后直接访问这个变量，而第一种情况是每次都遍历作用域链，直到全局环境，我们看到第二种情况实际上只遍历了一次，而第一种情况却是每次都遍历了，而且这种差别在多级作用域链和多个全局变量的情况下还会表现的非常明显。在作用域链查找的次数是`O(n)`。通过创建一个指向`document`的局部变量，就可以通过限制一次全局查找来改进这个函数的性能。

#### 避开闭包陷阱

闭包是个强大的工具，但同时也是性能问题的主要诱因之一。不合理的使用闭包会导致内存泄漏。闭包的性能不如使用内部方法，更不如重用外部方法。

- 由于`IE 9`浏览器的`DOM`节点作为`COM`对象来实现，`COM`的`内存管理`是通过引用计数的方式，引用计数有个难题就是循环引用，一旦`DOM`引用了闭包(例如`event handler`)，闭包的上层元素又引用了这个`DOM`，就会造成循环引用从而导致内存泄漏。

#### 原型优化

通过原型优化方法定义:如果一个方法类型将被频繁构造，通过方法原型从外面定义附加方法，从而避免方法的重复定义。可以通过外部原型的构造方式初始化值类型的变量定义。（这里强调值类型的原因是，引用类型如果在原型中定义，一个实例对引用类型的更改会影响到其他实例。）

- 这条规则中涉及到`JAVASCRIPT`中原型的概念，构造函数都有一个`prototype`属性，指向另一个对象。这个对象的所有属性和方法，都会被构造函数的实例继承。可以把那些不变的属性和方法，直接定义在`prototype`对象上。
	- 可以通过对象实例访问保存在原型中的值。
	- 不能通过对象实例重写原型中的值。
	- 在实例中添加一个与实例原型同名属性，那该属性就会屏蔽原型中的属性。
	- 通过delete操作符可以删除实例中的属性。
	
#### 变量优化
- 手工解除变量引用
	- 在业务代码中，一个变量已经确定不再需要了，那么就可以手工解除变量引用，以使其被回收。

			var data = { /* some big data */ };
			// ...
			data = null;
- 变量查找优化。
	- 变量声明带上`var`，如果声明变量忘记了`var`，那么`JAVASCRIPT`引擎将会遍历整个作用域查找这个变量，结果不管找到与否，都会造成性能损耗。
		- 如果在上级作用域找到了这个变量，上级作用域变量的内容将被无声的改写，导致莫名奇妙的错误发生。
		- 如果在上级作用域没有找到该变量，这个变量将自动被声明为全局变量，然而却都找不到这个全局变量的定义。
	- 慎用全局变量。
		- 全局变量需要搜索更长的作用域链。		
		- 全局变量的生命周期比局部变量长，不利于内存释放。	
		- 过多的全局变量容易造成混淆，增大产生bug的可能性。
		
> - 避免双重解释
>
	当`JAVASCRIPT`代码想解析`JAVASCRIPT`代码时就会存在双重解释惩罚，双重解释一般在使用`eval`函数、`new Function`构造函数和`setTimeout`传一个字符串时等情况下会遇到，如。
>
		eval("alert('hello world');");
		var sayHi = new Function("alert('hello world');");
		setTimeout("alert('hello world');", 100);
>    
     上述`alert('hello world');`语句包含在字符串中，即在JS代码运行的同时必须新启运一个解析器来解析新的代码，而实例化一个新的解析器有很大的性能损耗。
		我们看看下面的例子：
>
		var sum, num1 = 1, num2 = 2;
		/**效率低**/
	    for(var i = 0; i < 10000; i++){
	        var func = new Function("sum+=num1;num1+=num2;num2++;");
	        func();
			//eval("sum+=num1;num1+=num2;num2++;");
	    }
		/**效率高**/
	    for(var i = 0; i < 10000; i++){
	        sum+=num1;
	        num1+=num2;
	        num2++;
	    }
>
> - 原生方法更快
	- 只要有可能，使用原生方法而不是自已用JS重写。原生方法是用诸如C/C++之类的编译型语言写出来的，要比JS的快多了。
> - 最小化语句数
>
	JS代码中的语句数量也会影响所执行的操作的速度，完成多个操作的单个语句要比完成单个操作的多个语句块快。故要找出可以组合在一起的语句，以减来整体的执行时间。这里列举几种模式
>
	- 多个变量声明
>
			/**不提倡**/
			var i = 1;
			var j = "hello";
			var arr = [1,2,3];
			var now = new Date();
			/**提倡**/
			var i = 1,
			    j = "hello",
			    arr = [1,2,3],
			    now = new Date();
>
	- 插入迭代值
>
			/**不提倡**/
			var name = values[i];
			i++;
			/**提倡**/
			var name = values[i++];
>
	- 使用数组和对象字面量，避免使用构造函数Array(),Object()
>
			/**不提倡**/
			var a = new Array();
			a[0] = 1;
			a[1] = "hello";
			a[2] = 45;
			var o = new Obejct();
			o.name = "bill";
			o.age = 13;
			/**提倡**/
			var a = [1, "hello", 45];
			var o = {
			    name : "bill",
			    age : 13
			};
>
> - 避免使用属性访问方法
	- JavaScript不需要属性访问方法，因为所有的属性都是外部可见的。
	- 添加属性访问方法只是增加了一层重定向 ，对于访问控制没有意义。
>
		使用属性访问方法示例
>
			function Car() {
			   this .m_tireSize = 17;
			   this .m_maxSpeed = 250;
			   this .GetTireSize = Car_get_tireSize;
			   this .SetTireSize = Car_put_tireSize;
			}
>
			function Car_get_tireSize() {
			   return this .m_tireSize;
			}
>
			function Car_put_tireSize(value) {
			   this .m_tireSize = value;
			}
			var ooCar = new Car();
			var iTireSize = ooCar.GetTireSize();
			ooCar.SetTireSize(iTireSize + 1);
>
		直接访问属性示例
>
			function Car() {
			   this .m_tireSize = 17;
			   this .m_maxSpeed = 250;
			}
			var perfCar = new Car();
			var iTireSize = perfCar.m_tireSize;
			perfCar.m_tireSize = iTireSize + 1;
>
> - 减少使用元素位置操作
>
	- 一般浏览器都会使用增量reflow的方式将需要reflow的操作积累到一定程度然后再一起触发，但是如果脚本中要获取以下属性，那么积累的reflow将会马上执行，已得到准确的位置信息。
>
			offsetLeft
			offsetTop
			offsetHeight
			offsetWidth
			scrollTop/Left/Width/Height
			clientTop/Left/Width/Height
			getComputedStyle()

###类型转换优化
> 
- 把数字转换成字符串。
	- 应用`""+1`，效率是最高。
		- 性能上来说：`""+字符串`>`String()`>`.toString()`>`new String()`。
			- `String()`属于内部函数，所以速度很快。
			- `.toString()`要查询原型中的函数，所以速度略慢。
			- `new String()`最慢。
- 浮点数转换成整型。
	- 错误使用使用`parseInt()`。
		- `parseInt()`是用于将`字符串`转换成`数字`，而不是`浮点数`和`整型`之间的转换。
	- 应该使用`Math.floor()`或者`Math.round()`。
		- `Math`是内部对象，所以`Math.floor()`其实并没有多少查询方法和调用的时间，速度是最快的。