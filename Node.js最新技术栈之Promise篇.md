#Node.js最新技术栈之Promise篇
##内容包括

1.  回调函数Callbacks
2.  异步JavaScript
3.  Promise/a+
4.  生成器Generators/ yield
5.  Async/ await
![](https://dn-cnode.qbox.me/FgKu20kvFqHrkgpjbQxXkV1DmrG1)


<h1>0）痛点</h1>
<p>我们知道nodejs的最大的优点是高并发，适合适合I/O密集型应用</p>
<p><img src="//dn-cnode.qbox.me/FlPgjSEeYky4OEsdWokPlzTnQVln" alt="arch.png"></p>
<p>每一个函数都是异步，并发上去了，但痛苦也来了，比如</p>
<p>我现在要执行4个步骤，依次执行，于是就有下面的代码</p>
<pre class="prettyprint"><code><span class="pln">step1</span><span class="pun">(</span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">value1</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    step2</span><span class="pun">(</span><span class="pln">value1</span><span class="pun">,</span><span class="pln"> </span><span class="kwd">function</span><span class="pun">(</span><span class="pln">value2</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
        step3</span><span class="pun">(</span><span class="pln">value2</span><span class="pun">,</span><span class="pln"> </span><span class="kwd">function</span><span class="pun">(</span><span class="pln">value3</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
            step4</span><span class="pun">(</span><span class="pln">value3</span><span class="pun">,</span><span class="pln"> </span><span class="kwd">function</span><span class="pun">(</span><span class="pln">value4</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
                </span><span class="com">// Do something with value4</span><span class="pln">
            </span><span class="pun">});</span><span class="pln">
        </span><span class="pun">});</span><span class="pln">
    </span><span class="pun">});</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>这里只有4个，那如果更多呢？会不会崩溃，反正我会</p>
<p>有解决办法么？</p>
<p>答案是就是：promise/a+规范，下面我们看一下promise/a+规范</p>
<p>我们再来个伏笔，看一下链式写法</p>
<h1>链式写法</h1>
<p>先举个jQuery的链式写法例子</p>
<pre class="prettyprint"><code><span class="pln">$</span><span class="pun">(</span><span class="str">'#tab'</span><span class="pun">).</span><span class="pln">eq</span><span class="pun">(</span><span class="pln">$</span><span class="pun">(</span><span class="kwd">this</span><span class="pun">).</span><span class="pln">index</span><span class="pun">()).</span><span class="pln">show</span><span class="pun">().</span><span class="pln">siblings</span><span class="pun">().</span><span class="pln">hide</span><span class="pun">();</span></code></pre><p>链式写法的核心是：每个方法都返回this（自己看jquery源码）</p>
<p>下面举一个this的简单例子</p>
<pre class="prettyprint"><code><span class="kwd">var</span><span class="pln"> obj </span><span class="pun">=</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
  step1</span><span class="pun">:</span><span class="kwd">function</span><span class="pun">(){</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">'a'</span><span class="pun">);</span><span class="pln">
    </span><span class="kwd">return</span><span class="pln"> </span><span class="kwd">this</span><span class="pun">;</span><span class="pln">
  </span><span class="pun">},</span><span class="pln">
  step2</span><span class="pun">:</span><span class="kwd">function</span><span class="pun">(){</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">'b'</span><span class="pun">);</span><span class="pln">    
    </span><span class="kwd">return</span><span class="pln"> </span><span class="kwd">this</span><span class="pun">;</span><span class="pln">
  </span><span class="pun">},</span><span class="pln">
  step3</span><span class="pun">:</span><span class="kwd">function</span><span class="pun">(){</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">'c'</span><span class="pun">);</span><span class="pln">
    </span><span class="kwd">return</span><span class="pln"> </span><span class="kwd">this</span><span class="pun">;</span><span class="pln">
  </span><span class="pun">},</span><span class="pln">
  step4</span><span class="pun">:</span><span class="kwd">function</span><span class="pun">(){</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">'d'</span><span class="pun">);</span><span class="pln">
    </span><span class="kwd">return</span><span class="pln"> </span><span class="kwd">this</span><span class="pun">;</span><span class="pln">
  </span><span class="pun">}</span><span class="pln">
</span><span class="pun">}</span><span class="pln">

console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">'-----\n'</span><span class="pun">);</span><span class="pln">
obj</span><span class="pun">.</span><span class="pln">step1</span><span class="pun">().</span><span class="pln">step2</span><span class="pun">().</span><span class="pln">step3</span><span class="pun">();</span><span class="pln">
console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">'-----\n'</span><span class="pun">);</span><span class="pln">
obj</span><span class="pun">.</span><span class="pln">step4</span><span class="pun">().</span><span class="pln">step2</span><span class="pun">().</span><span class="pln">step1</span><span class="pun">();</span></code></pre><p>这只是一个简单返回this的链式写法而已</p>
<p>执行结果</p>
<pre class="prettyprint"><code><span class="pln">$ node doc</span><span class="pun">/</span><span class="kwd">this</span><span class="pun">.</span><span class="pln">js
</span><span class="pun">-----</span><span class="pln">

a
b
c
</span><span class="pun">-----</span><span class="pln">

c
b
a</span></code></pre><p>为啥要讲这个呢？先回到异步调用的场景，我们能这样调用么？如果每一个操作都可以连起来，是不是很爽</p>
<p>比如</p>
<pre class="prettyprint"><code><span class="kwd">return</span><span class="pln"> step1</span><span class="pun">().</span><span class="pln">step2</span><span class="pun">().</span><span class="pln">step3</span><span class="pun">().</span><span class="pln">step4</span><span class="pun">()</span></code></pre><p>这样做的好处</p>
<ul>
<li>1）每一个操作都是独立的函数</li>
<li>2）可组装，拼就好了</li>
</ul>
<p>我们还可以再苛刻点</p>
<ul>
<li>1）如果要是上一步的结果作为下一步的输入就更好了（linux里的pipe：ps -ef|grep node|awk ‘{print $2}’|xargs kill -9）</li>
<li>2）如果出错能捕获异常就更好了</li>
<li>3）如果能在函数里面也能控制流程就更好了</li>
</ul>
<p>会有人说：“你要求的是不是太多了？”</p>
<p>“but，很好，我们来制定一个叫promisesaplus的规范吧，涵盖上面的所有内容”</p>
<p>于是就有了promise/a+规范…</p>
<h1>1）promise/a+规范</h1>
<p>这里面讲很多链式相关的东西，是我们异步操作里常见的问题，我们希望有更好的解决方案，promise/a+规范便应运而生</p>
<h2>什么是promise/a+规范？</h2>
<p>Promise表示一个异步操作的最终结果。与Promise最主要的交互方法是通过将函数传入它的then方法从而获取得Promise最终的值或Promise最终最拒绝（reject）的原因。</p>
<p>想想我们刚才的那个例子，让我们来庖解一下</p>
<ul>
<li>a) 异步操作的最终结果</li>
</ul>
<p>简单点看，它也是一个链式操作,类似</p>
<pre class="prettyprint"><code><span class="kwd">return</span><span class="pln"> step1</span><span class="pun">().</span><span class="pln">step2</span><span class="pun">().</span><span class="pln">step3</span><span class="pun">().</span><span class="pln">step4</span><span class="pun">()</span></code></pre><ul>
<li>b) 与Promise最主要的交互方法是通过将函数传入它的then方法</li>
</ul>
<p>但它有点差别</p>
<pre class="prettyprint"><code><span class="kwd">return</span><span class="pln"> step1</span><span class="pun">().</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step2</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step3</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step4</span><span class="pun">)</span></code></pre><ul>
<li>c1) 从而获取得Promise最终的值或Promise最终最拒绝（reject）的原因</li>
</ul>
<p>比如step1执行成功了，那么就会执行step2，如果step2执行成功了就会执行下一个，以此类推</p>
<p>那么如果失败呢？于是应该改成这样：</p>
<pre class="prettyprint"><code><span class="kwd">return</span><span class="pln"> step1</span><span class="pun">().</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step2</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step3</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step4</span><span class="pun">).</span><span class="kwd">catch</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">err</span><span class="pun">){</span><span class="pln">
  </span><span class="com">// do something when err</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>加了一个catch error回调，当出现异常就会直接到这个流程，会不会很好呢？</p>
<ul>
<li>c2) 从而获取得Promise最终的值或Promise最终最拒绝（reject）的原因</li>
</ul>
<p>这句话了有一个reject，其实还对应一个resolve方法</p>
<ul>
<li>reject 是拒绝，跳转到catch error</li>
<li>resolve 是解决，下一步，即跳转到下一个promise操作</li>
</ul>
<p>我们还以上面的例子来说，如果step1失败了，走step3，step3失败了step4,如果step4出错就结束，报错，看一下代码</p>
<pre class="prettyprint"><code><span class="kwd">function</span><span class="pln"> step1</span><span class="pun">(){</span><span class="pln">
  </span><span class="kwd">if</span><span class="pun">(</span><span class="kwd">false</span><span class="pun">){</span><span class="pln">
    </span><span class="kwd">return</span><span class="pln"> step3</span><span class="pun">().</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step4</span><span class="pun">).</span><span class="kwd">catch</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">err</span><span class="pun">){</span><span class="pln">
      </span><span class="com">// do something when err</span><span class="pln">
    </span><span class="pun">});</span><span class="pln">
  </span><span class="pun">}</span><span class="pln">
  
</span><span class="pun">}</span><span class="pln">
</span><span class="kwd">return</span><span class="pln"> step1</span><span class="pun">().</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step2</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step3</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">step4</span><span class="pun">).</span><span class="kwd">catch</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">err</span><span class="pun">){</span><span class="pln">
  </span><span class="com">// do something when err</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>总结一下：以上就是promise/a+规范的定义，我们拆解了一下，涵盖了上面我提出的几个痛点</p>
<h2>术语</h2>
<h3>promise</h3>
<p>是一个包含了兼容promise规范then方法的对象或函数</p>
<p>我们可以这样理解，每一个promise只要返回的可以then的都可以。就像上面举例返回的this一样，只要每一个都返回this，她就可以无限的链式下去，</p>
<p>这里的this约定为每一个对象或函数返回的都是兼容promise规范then方法。</p>
<h3>thenable</h3>
<p>是一个包含了then方法的对象或函数。</p>
<p>这个可以这样理解，上面的例子，每个方法都返回this，比较麻烦，</p>
<p>那能不能只有then方法promise对象，每一个操作都返回一样的promise对象，是不是就可以无限链接下去了？</p>
<h3>value</h3>
<p>是任何Javascript值。 (包括 undefined, thenable, promise等).</p>
<p>promise/a+规范约定的几个概念而已</p>
<h3>exception</h3>
<p>是由throw表达式抛出来的值。</p>
<p>上面讲的，当流程出现异常的适合，把异常抛出来，由catch err处理。</p>
<h3>reason</h3>
<p>是一个用于描述Promise被拒绝原因的值。</p>
<p>就是给你一个犯了错误，自我交待，争取宽大处理的机会。</p>
<h2>参考</h2>
<p>还有一些promise里流程处理相关的东西，比较复杂，没啥大意义，有兴趣的可以看看官方网址</p>
<ul>
<li><a href="https://promisesaplus.com/" target="_blank">https://promisesaplus.com/</a></li>
</ul>
<h2>总结一下</h2>
<ul>
<li>
<ol>
<li>每个操作都返回一样的promise对象，保证链式操作</li>
</ol>
</li>
<li>
<ol>
<li>每个链式都通过then方法</li>
</ol>
</li>
<li>
<ol>
<li>每个操作内部允许犯错，出了错误，统一由catch error处理</li>
</ol>
</li>
<li>
<ol>
<li>操作内部，也可以是一个操作链，通过reject或resolve再造流程</li>
</ol>
</li>
</ul>
<p>理解这些，实际上nodejs里就没有啥难点了。当然我还要告诉更多好处，让你受益一生（吹吹牛）</p>
<h2>八卦一下</h2>
<p>哪个语言不需要回调呢？callback并不是js所特有的，举个例子oc里的block（如果愿意扯，也可以扯扯函数指针），如果用asi或者afnetworking处理http请求，你都可以选择用block方式，那么你如果一个业务里有多个请求呢？</p>
<p>你也只能嵌套，因为嵌套没有太多层，因为可以有其他解耦方式，其实我最想表达的是，它也可以使用promise方式的。</p>
<p>这里展示一个例子</p>
<pre class="prettyprint"><code><span class="pun">[</span><span class="kwd">self</span><span class="pln"> login</span><span class="pun">].</span><span class="kwd">then</span><span class="pun">(^{</span><span class="pln">

    </span><span class="com">// our login method wrapped an async task in a promise</span><span class="pln">
    </span><span class="kwd">return</span><span class="pln"> </span><span class="pun">[</span><span class="pln">API fetchKittens</span><span class="pun">];</span><span class="pln">

</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(^(</span><span class="typ">NSArray</span><span class="pln"> </span><span class="pun">*</span><span class="pln">fetchedKittens</span><span class="pun">){</span><span class="pln">

    </span><span class="com">// our API class wraps our API and returns promises</span><span class="pln">
    </span><span class="com">// fetchKittens returned a promise that resolves with an array of kittens</span><span class="pln">
    </span><span class="kwd">self</span><span class="pun">.</span><span class="pln">kittens </span><span class="pun">=</span><span class="pln"> fetchedKittens</span><span class="pun">;</span><span class="pln">
    </span><span class="pun">[</span><span class="kwd">self</span><span class="pun">.</span><span class="pln">tableView reloadData</span><span class="pun">];</span><span class="pln">

</span><span class="pun">}).</span><span class="kwd">catch</span><span class="pun">(^(</span><span class="typ">NSError</span><span class="pln"> </span><span class="pun">*</span><span class="pln">error</span><span class="pun">){</span><span class="pln">

    </span><span class="com">// any errors in any of the above promises land here</span><span class="pln">
    </span><span class="pun">[[[</span><span class="typ">UIAlertView</span><span class="pln"> alloc</span><span class="pun">]</span><span class="pln"> init</span><span class="pun">…]</span><span class="pln"> show</span><span class="pun">];</span><span class="pln">

</span><span class="pun">});</span><span class="pln"> </span></code></pre><p>详见http://promisekit.org/introduction/</p>
<p>是不是很奇怪，咋和上面的promise写法一样呢？</p>
<p>官方解释是</p>
<pre class="prettyprint"><code><span class="typ">PromiseKit</span><span class="pln"> </span><span class="kwd">is</span><span class="pln"> </span><span class="kwd">not</span><span class="pln"> just a promises implementation</span></code></pre><p>好吧，这只是冰山一角</p>
<p>如果各位熟悉前端js，相信你一定了解</p>
<ul>
<li>jQuery（1.5+）的deferred</li>
<li>Angularjs的$q对象</li>
</ul>
<p>nodejs里的实现</p>
<ul>
<li>bluebird （<a href="https://github.com/petkaantonov/bluebird" target="_blank">https://github.com/petkaantonov/bluebird</a> 后面继续讲，保持神秘）</li>
<li>q （<a href="https://github.com/kriskowal/q" target="_blank">https://github.com/kriskowal/q</a>  Angularjs的$q对象是q的精简版）</li>
<li>then （teambition作品  <a href="https://github.com/teambition/then.js" target="_blank">https://github.com/teambition/then.js</a> 没用过）</li>
<li>when  （<a href="https://github.com/cujojs/when" target="_blank">https://github.com/cujojs/when</a> 没用过）</li>
<li>async  （<a href="https://github.com/caolan/async" target="_blank">https://github.com/caolan/async</a> 最简单的）</li>
<li>eventproxy（朴灵作品 <a href="https://github.com/JacksonTian/eventproxy%EF%BC%8C%E4%BD%BF%E7%94%A8event%E6%9D%A5%E5%A4%84%E7%90%86%E6%B5%81%E7%A8%8B%EF%BC%8C%E4%B9%9F%E6%98%AF%E4%B8%8D%E9%94%99%E7%9A%84%E5%B0%9D%E8%AF%95%EF%BC%89" target="_blank">https://github.com/JacksonTian/eventproxy，使用event来处理流程，也是不错的尝试）</a></li>
</ul>
<p>其他语言实现，详见 <a href="https://promisesaplus.com/implementations" target="_blank">https://promisesaplus.com/implementations</a></p>
<p>其实，只要掌握了promise/a+规范，你就可以在n种语言里使用了</p>
<h1>2）如何实现</h1>
<p>promise/a+规范是一个通用解决方案，不只是对nodejs管用，只要你掌握了原理，就只是换个语言实现而已</p>
<p>下面我就带着大家看一下js里是如何实现的</p>
<pre class="prettyprint"><code><span class="kwd">var</span><span class="pln"> </span><span class="typ">Promise</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
</span><span class="pun">};</span><span class="pln">

</span><span class="kwd">var</span><span class="pln"> isPromise </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">value</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    </span><span class="kwd">return</span><span class="pln"> value </span><span class="kwd">instanceof</span><span class="pln"> </span><span class="typ">Promise</span><span class="pun">;</span><span class="pln">
</span><span class="pun">};</span><span class="pln">

</span><span class="kwd">var</span><span class="pln"> defer </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    </span><span class="kwd">var</span><span class="pln"> pending </span><span class="pun">=</span><span class="pln"> </span><span class="pun">[],</span><span class="pln"> value</span><span class="pun">;</span><span class="pln">
    </span><span class="kwd">var</span><span class="pln"> promise </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">new</span><span class="pln"> </span><span class="typ">Promise</span><span class="pun">();</span><span class="pln">
    promise</span><span class="pun">.</span><span class="kwd">then</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">callback</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
        </span><span class="kwd">if</span><span class="pln"> </span><span class="pun">(</span><span class="pln">pending</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
            pending</span><span class="pun">.</span><span class="pln">push</span><span class="pun">(</span><span class="pln">callback</span><span class="pun">);</span><span class="pln">
        </span><span class="pun">}</span><span class="pln"> </span><span class="kwd">else</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
            callback</span><span class="pun">(</span><span class="pln">value</span><span class="pun">);</span><span class="pln">
        </span><span class="pun">}</span><span class="pln">
    </span><span class="pun">};</span><span class="pln">
    </span><span class="kwd">return</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
        resolve</span><span class="pun">:</span><span class="pln"> </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">_value</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
            </span><span class="kwd">if</span><span class="pln"> </span><span class="pun">(</span><span class="pln">pending</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
                value </span><span class="pun">=</span><span class="pln"> _value</span><span class="pun">;</span><span class="pln">
                </span><span class="kwd">for</span><span class="pln"> </span><span class="pun">(</span><span class="kwd">var</span><span class="pln"> i </span><span class="pun">=</span><span class="pln"> </span><span class="lit">0</span><span class="pun">,</span><span class="pln"> ii </span><span class="pun">=</span><span class="pln"> pending</span><span class="pun">.</span><span class="pln">length</span><span class="pun">;</span><span class="pln"> i </span><span class="pun">&lt;</span><span class="pln"> ii</span><span class="pun">;</span><span class="pln"> i</span><span class="pun">++)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
                    </span><span class="kwd">var</span><span class="pln"> callback </span><span class="pun">=</span><span class="pln"> pending</span><span class="pun">[</span><span class="pln">i</span><span class="pun">];</span><span class="pln">
                    callback</span><span class="pun">(</span><span class="pln">value</span><span class="pun">);</span><span class="pln">
                </span><span class="pun">}</span><span class="pln">
                pending </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">undefined</span><span class="pun">;</span><span class="pln">
            </span><span class="pun">}</span><span class="pln">
        </span><span class="pun">},</span><span class="pln">
        promise</span><span class="pun">:</span><span class="pln"> promise
    </span><span class="pun">};</span><span class="pln">
</span><span class="pun">};</span></code></pre><p>首先</p>
<p>1） 声明promise对象</p>
<p>var promise = new Promise();</p>
<p>2）给promise对象增加then方法</p>
<p>promise.then = function (callback) {</p>
<p>3）给defer对象返回resolve和promise</p>
<p>4）value，在resolve事件里传参，是第几个就执行第几个</p>
<p>本来想讲的更多一点，但时间不允许啊（后面还有很多内容），此处暂时讲到这里</p>
<p>cnode里的William17写的挺好的，完整的实现可以参考</p>
<ul>
<li><a href="https://cnodejs.org/topic/5603cb8a152fdd025f0f5014" target="_blank">https://cnodejs.org/topic/5603cb8a152fdd025f0f5014</a></li>
<li><a href="https://github.com/William17/taxi" target="_blank">https://github.com/William17/taxi</a></li>
</ul>
<h2>关于Q</h2>
<p>q是一个不错的项目，也是比较早的promise实现，而且angularjs的$q就是它的精简版，如果掌握了q，学习angular和bluebird都会比较简单</p>
<p>当然它还有更棒的，它把q的7个版本是如何实现的都详细记录了，刚才我给出的v1实际上就q的早期版本</p>
<p><a href="https://github.com/kriskowal/q/tree/v1/design" target="_blank">https://github.com/kriskowal/q/tree/v1/design</a></p>
<p>很多人都以为是跟着人学，但人有不确定性，而且互联网让世界是平的了，我们如果能够学会从开源项目里学习，这才是长久的学习力</p>
<p>曾经写过一篇《如何学习之善用github篇：向 <a href="/user/Pana" target="_blank">@Pana</a> 学习》，有兴趣的可以去翻翻</p>
<p><a href="http://mp.weixin.qq.com/s?__biz=MzAxMTU0NTc4Nw==&amp;mid=223428929&amp;idx=1&amp;sn=8296f5a96aa34f836ab83000f7ad995b#rd" target="_blank">http://mp.weixin.qq.com/s?__biz=MzAxMTU0NTc4Nw==&amp;mid=223428929&amp;idx=1&amp;sn=8296f5a96aa34f836ab83000f7ad995b#rd</a></p>
<h1>3）真实项目里的实践</h1>
<h2>技术选项：先看基准测试</h2>
<p>2015-01-05 当时最新的模块，比较结果如下</p>
<p><img src="//dn-cnode.qbox.me/Fq-FM_N-zrlVEm-_7RS291Afo8dW" alt="benchmark.png"></p>
<p>对比一下结果</p>
<pre class="prettyprint"><code><span class="pun">顺序执行</span><span class="pln">
promises</span><span class="pun">-</span><span class="pln">bluebird</span><span class="pun">-</span><span class="pln">generator</span><span class="pun">.</span><span class="pln">js                </span><span class="lit">235</span><span class="pln">       </span><span class="lit">38.04</span><span class="pln">
promises</span><span class="pun">-</span><span class="pln">bluebird</span><span class="pun">.</span><span class="pln">js                          </span><span class="lit">335</span><span class="pln">       </span><span class="lit">52.08</span><span class="pln">

</span><span class="pun">并发执行</span><span class="pln">
promises</span><span class="pun">-</span><span class="pln">bluebird</span><span class="pun">.</span><span class="pln">js                     </span><span class="lit">389</span><span class="pln">       </span><span class="lit">53.49</span><span class="pln">
promises</span><span class="pun">-</span><span class="pln">bluebird</span><span class="pun">-</span><span class="pln">generator</span><span class="pun">.</span><span class="pln">js           </span><span class="lit">491</span><span class="pln">       </span><span class="lit">55.52</span></code></pre><p>不管是哪种，bluebird都是前三名。然后扒一扒第一名的<code>callbacks-baseline.js</code>,其实就我们最讨厌的最基本的回调写法，也就说，bb实际比原生js稍慢，但比其他promise库都快</p>
<p>综合一下bb的特性</p>
<ul>
<li>速度最快</li>
<li>api和文档完善，（对各个库支持都不错）</li>
<li>支持generator等未来发展趋势</li>
<li>github活跃</li>
<li>还有让人眼前一亮的功能点（保密，下面会讲）</li>
</ul>
<p>我们的结论只有bb是目前最好的最好的选择</p>
<p>我个人的一个小喜好，对caolan的async挺喜欢的，我的选项</p>
<ul>
<li>公司项目是bb</li>
<li>开源大点的还是bb</li>
<li>小工具啥的我会选async</li>
</ul>
<p>其实when，then，eventproxy等也不错，只是懒得折腾</p>
<p>大家在直播过程中有任何不明白或者想提问的可以随时私信给小助手，我会在答疑阶段统一回答</p>
<h2>场景</h2>
<p>下面举几个实际场景</p>
<h3>神器，bluebird的promisify</h3>
<p>promisify原理</p>
<p>就是你给他传一个对象或者prototype，它去遍历，给他们加上async方法，此方法返回promise对象，你就可以为所欲为了</p>
<ul>
<li>优点：使用简单</li>
<li>缺点：谨防对象过大，内存问题</li>
</ul>
<p>nodejs api支持</p>
<pre class="prettyprint"><code><span class="com">//Read more about promisification in the API Reference:</span><span class="pln">
</span><span class="com">//API.md</span><span class="pln">
</span><span class="kwd">var</span><span class="pln"> fs </span><span class="pun">=</span><span class="pln"> </span><span class="typ">Promise</span><span class="pun">.</span><span class="pln">promisifyAll</span><span class="pun">(</span><span class="kwd">require</span><span class="pun">(</span><span class="str">"fs"</span><span class="pun">));</span><span class="pln">

fs</span><span class="pun">.</span><span class="pln">readFileAsync</span><span class="pun">(</span><span class="str">"myfile.json"</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">JSON</span><span class="pun">.</span><span class="pln">parse</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">json</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">"Successful json"</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">catch</span><span class="pun">(</span><span class="typ">SyntaxError</span><span class="pun">,</span><span class="pln"> </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">e</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">error</span><span class="pun">(</span><span class="str">"file contains invalid json"</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">catch</span><span class="pun">(</span><span class="typ">Promise</span><span class="pun">.</span><span class="typ">OperationalError</span><span class="pun">,</span><span class="pln"> </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">e</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    console</span><span class="pun">.</span><span class="pln">error</span><span class="pun">(</span><span class="str">"unable to read file, because: "</span><span class="pun">,</span><span class="pln"> e</span><span class="pun">.</span><span class="pln">message</span><span class="pun">);</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>熟悉fs的API的都知道，fs有</p>
<ul>
<li>fs.readFile</li>
<li>fs.readFileSync</li>
</ul>
<p>但没有</p>
<ul>
<li>fs.readFileAsync</li>
</ul>
<p>实际上<code>fs.readFileAsync</code>是bluebird加上去的</p>
<pre class="prettyprint"><code><span class="kwd">var</span><span class="pln"> fs </span><span class="pun">=</span><span class="pln"> </span><span class="typ">Promise</span><span class="pun">.</span><span class="pln">promisifyAll</span><span class="pun">(</span><span class="kwd">require</span><span class="pun">(</span><span class="str">"fs"</span><span class="pun">));</span></code></pre><p>其实也没什么神奇的，只是bb做的更多而已，让调用足够简便</p>
<p>下面我们来看一下mvc里的model层如何使用bb，我们先假定使用的是mongoose啊</p>
<p>先定义模型</p>
<pre class="prettyprint"><code><span class="kwd">var</span><span class="pln"> mongoose </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">require</span><span class="pun">(</span><span class="str">'mongoose'</span><span class="pun">);</span><span class="pln">
</span><span class="kwd">var</span><span class="pln"> </span><span class="typ">Schema</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> mongoose</span><span class="pun">.</span><span class="typ">Schema</span><span class="pun">;</span><span class="pln">
</span><span class="kwd">var</span><span class="pln"> </span><span class="typ">Promise</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">require</span><span class="pun">(</span><span class="str">"bluebird"</span><span class="pun">);</span><span class="pln">

</span><span class="typ">UserSchema</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">new</span><span class="pln"> </span><span class="typ">Schema</span><span class="pun">({</span><span class="pln">
  username</span><span class="pun">:</span><span class="pln"> </span><span class="typ">String</span><span class="pun">,</span><span class="pln">
  password</span><span class="pun">:</span><span class="pln"> </span><span class="typ">String</span><span class="pun">,</span><span class="pln">
  created_at</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    type</span><span class="pun">:</span><span class="pln"> </span><span class="typ">Date</span><span class="pun">,</span><span class="pln">
    </span><span class="str">"default"</span><span class="pun">:</span><span class="pln"> </span><span class="typ">Date</span><span class="pun">.</span><span class="pln">now
  </span><span class="pun">}</span><span class="pln">
</span><span class="pun">});</span><span class="pln">

</span><span class="kwd">var</span><span class="pln"> </span><span class="typ">User</span><span class="pln"> </span><span class="pun">=</span><span class="pln"> mongoose</span><span class="pun">.</span><span class="pln">model</span><span class="pun">(</span><span class="str">'User'</span><span class="pun">,</span><span class="pln"> </span><span class="typ">UserSchema</span><span class="pun">);</span><span class="pln">

</span><span class="typ">Promise</span><span class="pun">.</span><span class="pln">promisifyAll</span><span class="pun">(</span><span class="typ">User</span><span class="pun">);</span><span class="pln">
</span><span class="typ">Promise</span><span class="pun">.</span><span class="pln">promisifyAll</span><span class="pun">(</span><span class="typ">User</span><span class="pun">.</span><span class="pln">prototype</span><span class="pun">);</span></code></pre><p>利用上面定义好的User模型，我们就可以写业务逻辑了</p>
<pre class="prettyprint"><code><span class="typ">User</span><span class="pun">.</span><span class="pln">findAsync</span><span class="pun">({</span><span class="pln">username</span><span class="pun">:</span><span class="pln"> username</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">data</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
   </span><span class="pun">...</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">catch</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">err</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
   </span><span class="pun">...</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>再来个复杂的</p>
<pre class="prettyprint"><code><span class="typ">TeamPartner</span><span class="pun">.</span><span class="pln">updateAsync</span><span class="pun">({</span><span class="pln">
	</span><span class="pun">...</span><span class="pln">
</span><span class="pun">},</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	$set</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
		</span><span class="pun">...</span><span class="pln">
	</span><span class="pun">}</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	</span><span class="kwd">return</span><span class="pln"> </span><span class="typ">User</span><span class="pun">.</span><span class="pln">findByIdAsync</span><span class="pun">(</span><span class="pln">user_id</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">team_owner</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	</span><span class="kwd">if</span><span class="pun">(</span><span class="kwd">typeof</span><span class="pln"> team_owner</span><span class="pun">.</span><span class="pln">partner_count </span><span class="pun">!=</span><span class="pln"> </span><span class="str">'undefined'</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
      team_new_count </span><span class="pun">=</span><span class="pln"> team_owner</span><span class="pun">.</span><span class="pln">partner_count </span><span class="pun">+</span><span class="pln"> </span><span class="lit">1</span><span class="pun">;</span><span class="pln">
  </span><span class="pun">}</span><span class="kwd">else</span><span class="pun">{</span><span class="pln">
      team_new_count </span><span class="pun">=</span><span class="pln"> </span><span class="lit">1</span><span class="pun">;</span><span class="pln">
  </span><span class="pun">}</span><span class="pln">

  </span><span class="com">// 创始人成员+1</span><span class="pln">
  </span><span class="kwd">return</span><span class="pln"> </span><span class="typ">User</span><span class="pun">.</span><span class="pln">findByIdAndUpdateAsync</span><span class="pun">(</span><span class="pln">user_id</span><span class="pun">,</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    $set</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
       </span><span class="pun">...</span><span class="pln">
    </span><span class="pun">}</span><span class="pln">
  </span><span class="pun">},</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
      upsert</span><span class="pun">:</span><span class="pln"> </span><span class="kwd">true</span><span class="pln">
  </span><span class="pun">});</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	</span><span class="kwd">return</span><span class="pln"> </span><span class="typ">Notify</span><span class="pun">.</span><span class="pln">findByIdAndUpdateAsync</span><span class="pun">(</span><span class="pln">notify_id</span><span class="pun">,</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
		$set</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
			read_status</span><span class="pun">:</span><span class="pln"> </span><span class="kwd">true</span><span class="pln">
		</span><span class="pun">}</span><span class="pln">
	</span><span class="pun">});</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
  </span><span class="kwd">var</span><span class="pln"> notifySave </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">new</span><span class="pln"> </span><span class="typ">Notify</span><span class="pun">({</span><span class="pln">
      </span><span class="pun">...</span><span class="pln">
  </span><span class="pun">});</span><span class="pln">

  </span><span class="kwd">return</span><span class="pln"> notifySave</span><span class="pun">.</span><span class="pln">saveAsync</span><span class="pun">();</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	</span><span class="kwd">return</span><span class="pln"> </span><span class="typ">User</span><span class="pun">.</span><span class="pln">findByIdAsync</span><span class="pun">(</span><span class="pln">user_join</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">joiner</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
  </span><span class="kwd">var</span><span class="pln"> teamSave </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">new</span><span class="pln"> </span><span class="typ">Team</span><span class="pun">({</span><span class="pln">
      </span><span class="pun">...</span><span class="pln">
  </span><span class="pun">});</span><span class="pln">
  </span><span class="kwd">return</span><span class="pln"> teamSave</span><span class="pun">.</span><span class="pln">saveAsync</span><span class="pun">();</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	res</span><span class="pun">.</span><span class="pln">status</span><span class="pun">(</span><span class="lit">200</span><span class="pun">).</span><span class="pln">json</span><span class="pun">({</span><span class="pln">
		data</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{},</span><span class="pln">
		status</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
			code</span><span class="pun">:</span><span class="pln"> </span><span class="lit">0</span><span class="pun">,</span><span class="pln">
			msg</span><span class="pun">:</span><span class="pln"> </span><span class="str">'success'</span><span class="pln">
		</span><span class="pun">}</span><span class="pln">
	</span><span class="pun">});</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">catch</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">err</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="pln">err</span><span class="pun">);</span><span class="pln">
	res</span><span class="pun">.</span><span class="pln">status</span><span class="pun">(</span><span class="lit">200</span><span class="pun">).</span><span class="pln">json</span><span class="pun">({</span><span class="pln">
		data</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{},</span><span class="pln">
		status</span><span class="pun">:</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
			code</span><span class="pun">:</span><span class="pln"> err</span><span class="pun">.</span><span class="pln">code</span><span class="pun">,</span><span class="pln">
			msg</span><span class="pun">:</span><span class="pln"> err</span><span class="pun">.</span><span class="pln">name
		</span><span class="pun">}</span><span class="pln">
	</span><span class="pun">});</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>很抱歉，伤害大家了，但确切是有这样用，我想问，脑子抽么？</p>
<p>优化方案</p>
<ul>
<li>mongoose上的static和method上扩展，别暴漏太多细节在控制层</li>
<li>面向promise，保证每个操作都是一个函数，让流程可以组装，不要忘了最初then的初衷</li>
</ul>
<pre class="prettyprint"><code><span class="kwd">function</span><span class="pln"> find_user</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	</span><span class="kwd">return</span><span class="pln"> </span><span class="typ">User</span><span class="pun">.</span><span class="pln">findByIdAsync</span><span class="pun">(</span><span class="pln">user_id</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}</span><span class="pln">

</span><span class="kwd">function</span><span class="pln"> find_user2</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
	</span><span class="kwd">return</span><span class="pln"> </span><span class="typ">User</span><span class="pun">.</span><span class="pln">findByIdAsync</span><span class="pun">(</span><span class="pln">user_id</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}</span><span class="pln">

</span><span class="kwd">function</span><span class="pln"> error</span><span class="pun">(</span><span class="pln">ex</span><span class="pun">){</span><span class="pln">
  console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="pln">ex</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}</span><span class="pln">

</span><span class="typ">TeamPartner</span><span class="pun">.</span><span class="pln">updateByXXAsync</span><span class="pun">(</span><span class="pln">a</span><span class="pun">,</span><span class="pln">b</span><span class="pun">,</span><span class="pln">c</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">find_user</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="pln">find_user2</span><span class="pun">).</span><span class="kwd">catch</span><span class="pun">(</span><span class="pln">error</span><span class="pun">)</span></code></pre><h2>开源项目里的promise</h2>
<ul>
<li>ioredis</li>
<li>mongoose &amp;&amp; mongoskin</li>
</ul>
<h3>ioredis</h3>
<p><a href="https://github.com/luin/ioredis" target="_blank">https://github.com/luin/ioredis</a></p>
<p>ioredis是redis库里的首选，其实它是基于node_redis的，增加和优化了很多，其中关于promise的一点是</p>
<pre class="prettyprint"><code><span class="typ">Delightful</span><span class="pln"> API</span><span class="pun">.</span><span class="pln"> </span><span class="typ">It</span><span class="pln"> works </span><span class="kwd">with</span><span class="pln"> </span><span class="typ">Node</span><span class="pln"> callbacks </span><span class="kwd">and</span><span class="pln"> </span><span class="typ">Bluebird</span><span class="pln"> promises</span><span class="pun">.</span></code></pre><p>用法</p>
<pre class="prettyprint"><code><span class="pln">redis</span><span class="pun">.</span><span class="kwd">get</span><span class="pun">(</span><span class="str">'foo'</span><span class="pun">).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(</span><span class="pln">result</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
  console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="pln">result</span><span class="pun">);</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>真心比node_redis好用的多。</p>
<p>luin是《Redis入门指南》一书作者，90后，全栈+设计，写过不少nodejs模块，目前就在群里</p>
<h3>mongoose &amp;&amp; mongoskin</h3>
<p>mongoose是nodejs处理mongodb的不二选择，简单说一下mongoskin</p>
<p>官方说</p>
<pre class="prettyprint"><code><span class="typ">The</span><span class="pln"> promise wrapper </span><span class="kwd">for</span><span class="pln"> node</span><span class="pun">-</span><span class="pln">mongodb</span><span class="pun">-</span><span class="kwd">native</span><span class="pun">.</span></code></pre><p>做了很多api上的优化，还是不错的，可是mongoose也出了promise</p>
<p>mongoose仿佛在喊：“用我，我也可以then”。。。。</p>
<h1>4）最后给出未来展望</h1>
<p>未来主要是es6和es7上的实现</p>
<ul>
<li>es6上的Generators/yield</li>
<li>es7上的Async/await</li>
</ul>
<p>都是好东西，可以尝试，毕竟早晚js还是要升级的，先做技术储备吧</p>
<h2>生成器Generators/yield</h2>
<p>生成器是ES6（也被称为ES2015）的新特性。</p>
<p>想象下面这样的一个场景：</p>
<pre class="prettyprint"><code><span class="pun">当你在执行一个函数的时候，你可以在某个点暂停函数的执行，并且做一些其他工作，然后再返回这个函数继续执行，</span><span class="pln"> </span><span class="pun">甚至是携带一些新的值，然后继续执行。</span></code></pre><p>上面描述的场景正是JavaScript生成器函数所致力于解决的问题。</p>
<p>当我们调用一个生成器函数的时候，它并不会立即执行， 而是需要我们手动的去执行迭代操作（next方法）。也就是说，你调用生成器函数，它会返回给你一个迭代器。迭代器会遍历每个中断点。</p>
<p>看例子</p>
<pre class="prettyprint"><code><span class="kwd">function</span><span class="pun">*</span><span class="pln"> foo </span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">  
  </span><span class="kwd">var</span><span class="pln"> index </span><span class="pun">=</span><span class="pln"> </span><span class="lit">0</span><span class="pun">;</span><span class="pln">
  </span><span class="kwd">while</span><span class="pln"> </span><span class="pun">(</span><span class="pln">index </span><span class="pun">&lt;</span><span class="pln"> </span><span class="lit">2</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    </span><span class="kwd">yield</span><span class="pln"> index</span><span class="pun">++;</span><span class="pln"> </span><span class="com">//暂停函数执行，并执行yield后的操作</span><span class="pln">
  </span><span class="pun">}</span><span class="pln">
</span><span class="pun">}</span><span class="pln">

</span><span class="kwd">var</span><span class="pln"> bar </span><span class="pun">=</span><span class="pln">  foo</span><span class="pun">();</span><span class="pln"> </span><span class="com">// 返回的其实是一个迭代器</span><span class="pln">

console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="pln">bar</span><span class="pun">.</span><span class="kwd">next</span><span class="pun">());</span><span class="pln">    </span><span class="com">// { value: 0, done: false }  </span><span class="pln">
console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="pln">bar</span><span class="pun">.</span><span class="kwd">next</span><span class="pun">());</span><span class="pln">    </span><span class="com">// { value: 1, done: false }  </span><span class="pln">
console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="pln">bar</span><span class="pun">.</span><span class="kwd">next</span><span class="pun">());</span><span class="pln">    </span><span class="com">// { value: undefined, done: true }  </span></code></pre><p>如果你想更轻松的使用生成器函数来编写异步JavaScript代码，我们可以使用 co 这个库，co是著名的tj大神写的，是一个为Node.js和浏览器打造的基于生成器的流程控制工具，借助于Promise，你可以使用更加优雅的方式编写非阻塞代码。</p>
<p>使用co，改写前面的示例代码：</p>
<pre class="prettyprint"><code><span class="pln">co</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">*</span><span class="pln"> </span><span class="pun">(){</span><span class="pln">  
  </span><span class="kwd">yield</span><span class="pln"> </span><span class="typ">Something</span><span class="pun">.</span><span class="pln">save</span><span class="pun">();</span><span class="pln">
</span><span class="pun">}).</span><span class="kwd">then</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">()</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
  </span><span class="com">// success</span><span class="pln">
</span><span class="pun">})</span><span class="pln">
</span><span class="pun">.</span><span class="kwd">catch</span><span class="pun">(</span><span class="kwd">function</span><span class="pun">(</span><span class="pln">err</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
  </span><span class="com">//error handling</span><span class="pln">
</span><span class="pun">});</span></code></pre><p>你可能会问：如何实现并行操作呢？答案可能比你想象的简单，如下</p>
<pre class="prettyprint"><code><span class="kwd">yield</span><span class="pln"> </span><span class="pun">[</span><span class="pln">a</span><span class="pun">.</span><span class="pln">save</span><span class="pun">(),</span><span class="pln"> b</span><span class="pun">.</span><span class="pln">save</span><span class="pun">()];</span><span class="pln">  </span></code></pre><p>其实它就是Promise.all，换种写法而已</p>
<h2>Async/await</h2>
<p>在ES7（还未正式标准化）中引入了Async函数的概念，目前如果你想要使用的话，只能借助于babel 这样的语法转换器将其转为ES5代码。</p>
<p>使用async关键字，可以轻松地达成之前使用生成器和co函数所做到的工作。当然，hack除外</p>
<p>可以这样理解，async函数实际使用的是Promise，然后使用await来执行异步操作，这里的await类似于上面讲的yield</p>
<p>因此，使用async重写上面的代码：</p>
<pre class="prettyprint"><code><span class="pln">async </span><span class="kwd">function</span><span class="pln"> save</span><span class="pun">(</span><span class="typ">Something</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">  
  </span><span class="kwd">try</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    await </span><span class="typ">Something</span><span class="pun">.</span><span class="pln">save</span><span class="pun">();</span><span class="pln"> </span><span class="com">// 等待await后面的代码执行完，类似于yield</span><span class="pln">
  </span><span class="pun">}</span><span class="pln"> </span><span class="kwd">catch</span><span class="pln"> </span><span class="pun">(</span><span class="pln">err</span><span class="pun">)</span><span class="pln"> </span><span class="pun">{</span><span class="pln">
    </span><span class="com">//error handling</span><span class="pln">
  </span><span class="pun">}</span><span class="pln">
  console</span><span class="pun">.</span><span class="pln">log</span><span class="pun">(</span><span class="str">'success'</span><span class="pun">);</span><span class="pln">
</span><span class="pun">}</span></code></pre><p>下一代web框架Koa也支持async函数，如果你也在使用koa，那么你现在就可以借助babel使用这一特性了</p>
<pre class="prettyprint"><code><span class="kwd">import</span><span class="pln"> koa </span><span class="kwd">from</span><span class="pln"> koa</span><span class="pun">;</span><span class="pln">  
let app </span><span class="pun">=</span><span class="pln"> koa</span><span class="pun">();</span><span class="pln">

app</span><span class="pun">.</span><span class="pln">experimental </span><span class="pun">=</span><span class="pln"> </span><span class="kwd">true</span><span class="pun">;</span><span class="pln">

app</span><span class="pun">.</span><span class="kwd">use</span><span class="pun">(</span><span class="pln">async </span><span class="kwd">function</span><span class="pln"> </span><span class="pun">(){</span><span class="pln">  
    </span><span class="kwd">this</span><span class="pun">.</span><span class="pln">body </span><span class="pun">=</span><span class="pln"> await </span><span class="typ">Promise</span><span class="pun">.</span><span class="pln">resolve</span><span class="pun">(</span><span class="str">'Hello Reader!'</span><span class="pun">)</span><span class="pln">
</span><span class="pun">})</span><span class="pln">

app</span><span class="pun">.</span><span class="pln">listen</span><span class="pun">(</span><span class="lit">3000</span><span class="pun">);</span><span class="pln">  </span></code></pre><p>这里报个料，群里<a href="https://github.com/fundon" target="_blank">fundon</a> 大神写的https://github.com/trekjs/trek
是基于koa的，使用babel编译，并有更多相当棒的特性。爱折腾的可以试试</p>
<p>最后还是要归纳总结一下，</p>
<p>先复习promise/a+的四个要点</p>
<ul>
<li>a) 异步操作的最终结果，尽可能每一个异步操作都是独立操作单元</li>
<li>b) 与Promise最主要的交互方法是通过将函数传入它的then方法（thenable）</li>
<li>c) 捕获异常catch error</li>
<li>d) 根据reject和resolve重塑流程</li>
</ul>
<p>已经第三遍，该记住了！</p>
<p>继续看第一个图</p>
<p><img src="//dn-cnode.qbox.me/FgKu20kvFqHrkgpjbQxXkV1DmrG1" alt="1.png"></p>
<p>再来看这张图，从callback到promose是历史的必然产物</p>
<p>generator是一种新的定义方式，定义操作单元，尤其在迭代器的情况，搭配yield来执行，可读性上差了很多，好处是真的解耦了</p>
<p>co是一个中间产品，可以说是给generator增加了promise实现，可读性和易用性是愿意好于generator + yield的</p>
<p>最后我们看看async，它实际上是通过async这个关键词，定义的函数就可以返回promise对象，可以说async就是能返回promise对象的generator。yield关键词以及被generator绑架了，那它就换个名字，叫await</p>
<p>其实从这段历史来看，反复就是promise上的折腾，只是加了generator这个别名，只是async是能返回promise的generator</p>
<p>这样理解是不是更简单呢？</p>
<p>万变不离其宗，想精通js或者nodejs，promise是大家必须迈过去的坑。</p>
<p>谢谢大家，今天讲的内容就到这里，如果有什么讲的不对的、不合理的，请不吝赐教，共同学习</p>
<p>大家有任何不明白或者想提问的可以随时私信给小助手，我会在答疑阶段统一回答</p>
<h2>推荐资料</h2>
<ul>
<li><a href="http://liubin.github.io/promises-book/#introduction" target="_blank">http://liubin.github.io/promises-book/#introduction</a></li>
<li><a href="http://www.mattgreer.org/articles/promises-in-wicked-detail/" target="_blank">http://www.mattgreer.org/articles/promises-in-wicked-detail/</a></li>
</ul>
<p>全文完</p>
<p>转自：https://cnodejs.org/topic/560dbc826a1ed28204a1e7de</p>
</div>
