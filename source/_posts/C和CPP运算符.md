---
title: C和CPP运算符
date: 2022-07-10 15:53:01
tags:
- CPP
- C语言
- VSCode
- MingGW64
---

下列运算符在两个语言中都是顺序点（运算符未重载时）： `&&`、`||`、`?:` 和 `,`（逗号运算符）。

C++也包含类型转换运算符 `const_cast`、`static_cast`、`dynamic_cast`和 `reinterpret_cast`，不在表中列出以维持简洁。类型转换运算符需要在表达式中明确使用括号，因此并不存在优先级的问题。

在C里有的运算符，除了逗号运算符和箭头记头的运算符以外，在Java、Perl、C#和PHP同样也有相同的优先级、结合性和语义。
<!--more-->
<table class="wikitable">
<tbody>
<tr><th>优先级</th><th>运算符</th><th>叙述</th><th>示例</th><th>重载性</th><th>结合性</th></tr>
<tr>
<td><code>1</code></td>
<td><code>::</code></td>
<td>作用域解析(C++专有)</td>
<td><code>Class::age = 2;</code></td>
<td>否</td>
<td rowspan="13">由左至右</td>
</tr>
<tr>
<td rowspan="12"><code>2</code></td>
<td><code>++</code></td>
<td>后缀递增</td>
<td>i++</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>--</code></td>
<td>后缀递减</td>
<td>i--</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>{}</code></td>
<td>组合</td>
<td>{i++;a*=i;}</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>()</code></td>
<td>函数调用或变量初始化</td>
<td>c_tor(int x, int y)&nbsp;: _x(x), _y(y * 10) {}</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>[]</code></td>
<td>数组访问</td>
<td>array[4] = 2;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>.</code></td>
<td>以对象方式访问成员</td>
<td>obj.age = 34;</td>
<td>否</td>
</tr>
<tr>
<td><code>-&gt;</code></td>
<td>以指针方式访问成员</td>
<td>ptr-&gt;age = 34;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>dynamic_cast</code></td>
<td>运行时检查类型转换(C++专有)</td>
<td>Y&amp; y = dynamic_cast&lt;Y&amp;&gt;(x);</td>
<td>否</td>
</tr>
<tr>
<td><code>static_cast</code></td>
<td>未经检查的类型转换(C++专有)</td>
<td>Y&amp; y = static_cast&lt;Y&amp;&gt;(x);</td>
<td>否</td>
</tr>
<tr>
<td><code>reinterpret_cast</code></td>
<td>重定义类型转换(C++专有)</td>
<td>int const* p = reinterpret_cast&lt;int const*&gt;(0x1234);</td>
<td>否</td>
</tr>
<tr>
<td><code>const_cast</code></td>
<td>更改非常量属性(C++专有)</td>
<td>int* q = const_cast&lt;int*&gt;(p);</td>
<td>否</td>
</tr>
<tr>
<td><code>typeid</code></td>
<td>获取类型信息(C++专有)</td>
<td>std::type_info const&amp; t = typeid(x);</td>
<td>否</td>
</tr>
<tr>
<td rowspan="14"><code>3</code></td>
<td><code>++</code></td>
<td>前缀递增</td>
<td>++i</td>
<td>&nbsp;</td>
<td rowspan="14">由右至左</td>
</tr>
<tr>
<td><code>--</code></td>
<td>前缀递减</td>
<td>--i</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>+</code></td>
<td>一元正号</td>
<td>int i = +1;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>-</code></td>
<td>一元负号</td>
<td>int i = -1;</td>
<td>&nbsp;</td>
</tr>
<tr>
<td><code>!</code><br><code>not</code></td>
<td>逻辑非<br><code>!</code>的备用拼写</td>
<td>if (!done) …</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>~</code><br><code>compl</code></td>
<td>按位取反<br><code>~</code>的备用拼写</td>
<td>flag1 = ~flag2;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>(<em>type</em>)</code></td>
<td>强制类型转换</td>
<td>int i = (int)floatNum;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>*</code></td>
<td>取指针指向的值</td>
<td>int data = *intPtr;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&amp;</code></td>
<td>取变量的地址</td>
<td>int *intPtr = &amp;data;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>sizeof</code></td>
<td>某某的大小</td>
<td>size_t s = sizeof(int);</td>
<td>否</td>

</tr>
<tr>
<td><code>new</code></td>
<td>动态内存分配(C++专有)</td>
<td>long* pVar = new long;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>new[]</code></td>
<td>动态数组内存分配(C++专有)</td>
<td>long* array = new long[20];</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>delete</code></td>
<td>动态内存释放(C++专有)</td>
<td>delete pVar;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>delete[]</code></td>
<td>动态数组内存释放(C++专有)</td>
<td>delete [] array;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td rowspan="2"><code>4</code></td>
<td><code>.*</code></td>
<td>成员对象选择(C++专有)</td>
<td>obj.*var = 24;</td>
<td>否</td>
<td rowspan="20">由左至右</td>

</tr>
<tr>
<td><code>-&gt;*</code></td>
<td>成员指针选择(C++专有)</td>
<td>ptr-&gt;*var = 24;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td rowspan="3"><code>5</code></td>
<td><code>*</code></td>
<td>乘法</td>
<td>int i = 2 * 4;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>/</code></td>
<td>除法</td>
<td>float f = 10.0 / 3.0;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>%</code></td>
<td>模数(取余)</td>
<td>int rem = 4&nbsp;% 3;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td rowspan="2"><code>6</code></td>
<td><code>+</code></td>
<td>加法</td>
<td>int i = 2 + 3;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>-</code></td>
<td>减法</td>
<td>int i = 5 - 1;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td rowspan="2"><code>7</code></td>
<td><code>&lt;&lt;</code></td>
<td>比特左移</td>
<td>int flags = 33 &lt;&lt; 1;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&gt;&gt;</code></td>
<td>比特右移</td>
<td>int flags = 33 &gt;&gt; 1;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td rowspan="4"><code>8</code></td>
<td><code>&lt;</code></td>
<td>小于关系</td>
<td>if (i &lt; 42) …</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&lt;=</code></td>
<td>小于等于关系</td>
<td>if (i &lt;= 42) ...</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&gt;</code></td>
<td>大于关系</td>
<td>if (i &gt; 42) …</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&gt;=</code></td>
<td>大于等于关系</td>
<td>if (i &gt;= 42) ...</td>
<td>&nbsp;</td>

</tr>
<tr>
<td rowspan="2"><code>9</code></td>
<td><code>==</code><br><code>eq</code></td>
<td>等于关系<br><code>==</code>的备用拼写</td>
<td>if (i == 42) ...</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>!=</code><br><code>not_eq</code></td>
<td>不等于关系<br><code>!=</code>的备用拼写</td>
<td>if (i&nbsp;!= 42) …</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>10</code></td>
<td><code>&amp;</code><br><code>bitand</code></td>
<td>比特 AND<br><code>&amp;</code>的备用拼写</td>
<td>flag1 = flag2 &amp; 42;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>11</code></td>
<td><code>^</code><br><code>xor</code></td>
<td>比特 XOR(独占or)<br><code>^</code>的备用拼写</td>
<td>flag1 = flag2 ^ 42;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>12</code></td>
<td><code>|</code><br><code>bitor</code></td>
<td>比特 OR(包含or)<br><code>|</code>的备用拼写</td>
<td>flag1 = flag2 | 42;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>13</code></td>
<td><code>&amp;&amp;</code><br><code>and</code></td>
<td>逻辑 AND<br><code>&amp;&amp;</code>的备用拼写</td>
<td>if (conditionA &amp;&amp; conditionB) …</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>14</code></td>
<td><code>||</code><br><code>or</code></td>
<td>逻辑 OR<br><code>||</code>的备用拼写</td>
<td>if (conditionA || conditionB) ...</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>15</code></td>
<td><code><em>c</em>?<em>t</em>:<em>f</em></code></td>
<td>三元条件运算</td>
<td>int i = a &gt; b&nbsp;? a&nbsp;: b;</td>
<td>否</td>
<td rowspan="12">由右至左</td>

</tr>
<tr>
<td rowspan="11"><code>16</code></td>
<td><code>=</code></td>
<td>直接赋值</td>
<td>int a = b;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>+=</code></td>
<td>以和赋值</td>
<td>a += 3;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>-=</code></td>
<td>以差赋值</td>
<td>b -= 4;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>*=</code></td>
<td>以乘赋值</td>
<td>a *= 5;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>/=</code></td>
<td>以除赋值</td>
<td>a /= 2;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>%=</code></td>
<td>以取余数赋值</td>
<td>a&nbsp;%= 3;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&lt;&lt;=</code></td>
<td>以比特左移赋值</td>
<td>flags &lt;&lt;= 2;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&gt;&gt;=</code></td>
<td>以比特右移赋值</td>
<td>flags &gt;&gt;= 2;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>&amp;=</code><br><code>and_eq</code></td>
<td>以比特AND赋值<br><code>&amp;=</code>的备用拼写</td>
<td>flags &amp;= new_flags;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>^=</code><br><code>xor_eq</code></td>
<td>以比特XOR赋值<br><code>^=</code>的备用拼写</td>
<td>flags ^= new_flags;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>|=</code><br><code>or_eq</code></td>
<td>以比特OR赋值<br><code>|=</code>的备用拼写</td>
<td>flags |= new_flags;</td>
<td>&nbsp;</td>

</tr>
<tr>
<td><code>17</code></td>
<td><code>throw</code></td>
<td>抛出异常</td>
<td>throw EClass(“Message”);</td>
<td>否</td>

</tr>
<tr>
<td><code>18</code></td>
<td><code>,</code></td>
<td>逗号运算符</td>
<td>for (i = 0, j = 0; i &lt; 10; i++, j++) …</td>
<td>&nbsp;</td>
<td>由左至右</td>
</tr>
</tbody>
</table>

## 列表

在本表中，a、b和c代表有效值（来自变量或返回值的逐字常数或数值）、对象名称，或适当的左值。

<table class="wikitable">
<tbody>
<tr>
<td colspan="4">
<h3 style="text-align: left"><span class="mw-headline">算术运算符</span></h3>

</td>

</tr>
<tr><th>运算符名称</th><th>语法</th><th>可重载</th><th><a title="C" href="http://zh.wikipedia.org/wiki/C" rel="noopener">C</a>里有</th></tr>
<tr>
<td>一元正号</td>
<td><code><strong>+</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>加法（总和）</td>
<td><code>a&nbsp;<strong>+</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>前缀递增</td>
<td><code><strong>++</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>后缀递增</td>
<td><code>a<strong>++</strong></code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以加法赋值</td>
<td><code>a&nbsp;<strong>+=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>一元负号（取反）</td>
<td><code><strong>-</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>减法（差）</td>
<td><code>a&nbsp;<strong>-</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>前缀递减</td>
<td><code><strong>--</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>后缀递减</td>
<td><code>a<strong>--</strong></code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以减法赋值</td>
<td><code>a&nbsp;<strong>-=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>乘法（乘积）</td>
<td><code>a&nbsp;<strong>*</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以乘法赋值</td>
<td><code>a&nbsp;<strong>*=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>除法（分之）</td>
<td><code>a&nbsp;<strong>/</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以除法赋值</td>
<td><code>a&nbsp;<strong>/=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>模数（余数）</td>
<td><code>a&nbsp;<strong>%</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以模数赋值</td>
<td><code>a&nbsp;<strong>%=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td colspan="4">
<h3><span class="mw-headline">比较运算符</span></h3>

</td>

</tr>
<tr><th>运算符名称</th><th>语法</th><th>可重载</th><th><a title="C" href="http://zh.wikipedia.org/wiki/C" rel="noopener">C</a>里有</th></tr>
<tr>
<td>小于</td>
<td><code>a&nbsp;<strong>&lt;</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>小于或等于</td>
<td><code>a&nbsp;<strong>&lt;=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>大于</td>
<td><code>a&nbsp;<strong>&gt;</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>大于或等于</td>
<td><code>a&nbsp;<strong>&gt;=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>不等于</td>
<td><code>a&nbsp;<strong>!=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>等于</td>
<td><code>a&nbsp;<strong>==</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>逻辑取反</td>
<td><code><strong>!</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>逻辑 AND</td>
<td><code>a&nbsp;<strong>&amp;&amp;</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>逻辑 OR</td>
<td><code>a&nbsp;<strong>||</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td colspan="4">
<h3><span class="mw-headline">比特运算符</span></h3>

</td>

</tr>
<tr><th>运算符名称</th><th>语法</th><th>可重载</th><th><a title="C" href="http://zh.wikipedia.org/wiki/C" rel="noopener">C</a>里有</th></tr>
<tr>
<td>比特左移</td>
<td><code>a&nbsp;<strong>&lt;&lt;</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以比特左移赋值</td>
<td><code>a&nbsp;<strong>&lt;&lt;=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>比特右移</td>
<td><code>a&nbsp;<strong>&gt;&gt;</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以比特右移赋值</td>
<td><code>a&nbsp;<strong>&gt;&gt;=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>比特一的补数</td>
<td><code><strong>~</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>比特 AND</td>
<td><code>a&nbsp;<strong>&amp;</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以比特 AND 赋值</td>
<td><code>a&nbsp;<strong>&amp;=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>比特 OR</td>
<td><code>a&nbsp;<strong>|</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以比特 OR 赋值</td>
<td><code>a&nbsp;<strong>|=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>比特 XOR</td>
<td><code>a&nbsp;<strong>^</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>以比特 XOR 赋值</td>
<td><code>a&nbsp;<strong>^=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td colspan="4">
<h3><span class="mw-headline">其它运算符</span></h3>

</td>

</tr>
<tr><th>运算符名称</th><th>语法</th><th>可重载</th><th><a title="C" href="http://zh.wikipedia.org/wiki/C" rel="noopener">C</a>里有</th></tr>
<tr>
<td>基本<a class="mw-redirect" title="C++赋值运算符" href="http://zh.wikipedia.org/wiki/C%2B%2B%E8%A8%AD%E5%AE%9A%E9%81%8B%E7%AE%97%E5%AD%90" rel="noopener">赋值</a></td>
<td><code>a&nbsp;<strong>=</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>函数调用</td>
<td><code>a<strong>()</strong></code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>数组下标</td>
<td><code>a<strong>[</strong>b<strong>]</strong></code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>间接（向下参考）</td>
<td><code><strong>*</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>的地址（参考）</td>
<td><code><strong>&amp;</strong>a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>成员指针</td>
<td><code>a<strong>-&gt;</strong>b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>成员</td>
<td><code>a<strong>.</strong>b</code></td>
<td class="table-no">否</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>间接成员指针</td>
<td><code>a<strong>-&gt;*</strong>b</code></td>
<td class="table-yes">是</td>
<td class="table-no">否</td>

</tr>
<tr>
<td>间接成员</td>
<td><code>a<strong>.*</strong>b</code></td>
<td class="table-no">否</td>
<td class="table-no">否</td>

</tr>
<tr>
<td>转换</td>
<td><code>(<em><a class="mw-redirect" title="資料型別" href="http://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%9E%8B%E5%88%A5" rel="noopener">type</a></em>) a</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>逗号</td>
<td><code>a&nbsp;<strong>,</strong>&nbsp;b</code></td>
<td class="table-yes">是</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>三元条件</td>
<td><code>a&nbsp;<strong>?</strong>&nbsp;b&nbsp;<strong>:</strong>&nbsp;c</code></td>
<td class="table-no">否</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>作用域解析</td>
<td><code>a<strong>::</strong>b</code></td>
<td class="table-no">否</td>
<td class="table-no">否</td>

</tr>
<tr>
<td>的大小</td>
<td><code><strong>sizeof</strong>&nbsp;a</code></td>
<td class="table-no">否</td>
<td class="table-yes">是</td>

</tr>
<tr>
<td>类型识别</td>
<td><code><strong>typeid</strong>&nbsp;<em><a class="mw-redirect" title="資料型別" href="http://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%9E%8B%E5%88%A5" rel="noopener">type</a></em></code></td>
<td class="table-no">否</td>
<td class="table-no">否</td>

</tr>
<tr>
<td>分配存储区</td>
<td><code><strong>new</strong>&nbsp;<em><a class="mw-redirect" title="資料型別" href="http://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%9E%8B%E5%88%A5" rel="noopener">type</a></em></code></td>
<td class="table-yes">是</td>
<td class="table-no">否</td>

</tr>
<tr>
<td>解除分配存储区</td>
<td><code><strong>delete</strong>&nbsp;a</code></td>
<td class="table-yes">是</td>
<td class="table-no">否</td>

</tr>

</tbody>

</table>
<h3><span class="mw-headline">语言扩展</span></h3>
<table class="wikitable">
<tbody>
<tr><th>运算符名称</th><th>语法</th><th>可重载</th><th>C里有</th><th>提供者</th></tr>
<tr>
<td><a class="new" title="Label Value Operator（页面不存在）" href="http://zh.wikipedia.org/w/index.php?title=Label_Value_Operator&amp;action=edit&amp;redlink=1" rel="noopener">标签值</a></td>
<td><code><strong>&amp;&amp;</strong>&nbsp;label</code></td>
<td class="table-no">否</td>
<td class="table-yes">是</td>
<td>GCC</td>

</tr>
<tr>
<td>取得型态</td>
<td><code><strong>typeof</strong>&nbsp;a</code><br><code><strong>typeof</strong>(<em>expr</em>)</code></td>
<td class="table-no">否</td>
<td class="table-yes">是</td>
<td>GCC</td>

</tr>
<tr>
<td>最小／最大值</td>
<td><code>a&nbsp;<strong>&lt;?</strong>&nbsp;b</code><br><code>a&nbsp;<strong>&gt;?</strong>&nbsp;b</code></td>
<td class="table-no">否</td>
<td class="table-no">否</td>
<td>GCC</td>
</tr>
</tbody>
</table>

参考：

[C和C++运算符](https://www.cnblogs.com/yangjig/p/3923515.html)