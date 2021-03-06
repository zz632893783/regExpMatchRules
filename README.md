# regExpMatchRules
正则表达式匹配规则

正则的总结，并不涉及到通配符含义，只是对正则匹配的规则总结了下，不敢保证一定对，只当自己整理的笔记

## 1.首先字符串中的每一个字符都有位置

例如字符串：abcde
这个时候位置关系如下
```
     a      b      c      d      e
  ↑      ↑      ↑      ↑      ↑      ↑
 0位置  1位置  2位置   3位置  4位置  5位置
```

## 2.正则表达式有一个占位的概念

分为两种①是普通匹配，②是断言，普通匹配占宽度，断言不占宽度，普通匹配匹配的是字符，断言匹配的是某一个位置，所以断言也被叫做零宽度断言。

例如表达式：/\w(?=\d)/
表达式中\w匹配一个字符，占用一个宽度，(?=\d)表示一个数字的位置，整个正则表达式连接起来就是，首先匹配一个字母或者数字 \w，匹配的这个 \w 后面必须紧跟一个数字(?=\d)，这里的 \w 为普通匹配，占用宽度，而 (?=\d) 为断言，并不占用宽度，只是规定 \w 后面必须含有一个数字

看一下使用上面这个表达书进行匹配的结果
```javascript
('abc123').match(/\w(?=\d)/);
// 匹配到的结果为 ['c', index: 2, input: 'abc123']
// 断言由于不占宽度所以match的结果只有字符'c'
```

## 3.正则表达式的控制权概念

联系第1点和第2点
例如我们有一个正则表达式，和一个需要进行匹配的字符串
```javascript
let reg = /bc/g;
let str = 'abcdabc';
```
```
      a      b      c      d      a      b      c
  ↑      ↑      ↑      ↑      ↑      ↑      ↑      ↑
0位置   1位置  2位置   3位置  4位置  5位置   6位置  7位置
```

1) 表达式 /bc/ 匹配bc字符，此时b是正则的起始匹配规则，整个正则控制权在b手中，首先从0位置开始匹配字符串'abcdabc'<br/>
2) 0位置之后字符为a，匹配失败，匹配位置由0位置转移到1位置，正则控制权回到整个正则表达式的最前面，所以这个时候控制权依旧在b手中(其实这里还牵扯到回溯，这里暂时先不做讨论)<br/>
3) 匹配位置转移到1之后，从1位置开始匹配，字符串1位置之后的字符为b,此时正则的控制权也是在b字符上，匹配成功，匹配的位置从1变为2，由于正则表达式中的b字符没有做量词修饰，即它表示只占有一个位置的宽度，所以b的控制结束，控制权移交给正则的下一个匹配字符c<br/>
4) 从2位置继续匹配，字符串2位置之后的字符为c，此时正则的控制权也正好在字符c上，匹配成功，匹配位置从2变为3，由于c没有做任何量词的修饰，表示他只占有1个宽度，所以c的控制结束，控制权继续移交给正则的下一个匹配字符<br/>
5) 匹配字符c之后已经没有其他的匹配字符了，整个正则表达式结束，此时字符串的匹配位置依旧是上一步结束时候的3位置<br/>
6) 由于正则表达式声明为g全局匹配，所以整个正则表达式将从头开始，控制权移交给整个正则表达式的起始字符b<br/>
7) 正则表达式控制权在b，匹配位置为之前匹配成功的结束位置3，字符串3位置之后的字符为d，表达式b匹配字符d失败，控制权依旧在起始位置b，字符串匹配位置由3变为4<br/>
8) 正则表达式控制权在b，匹配位置为4，字符串4位置时候的字符为a，表达式b匹配字符a失败，控制权依旧在起始位置b，字符串匹配位置由4变为5<br/>
9) 正则表达式控制权在b，匹配位置为5，字符串5位置时候的字符为b，匹配成功，匹配的位置从5变为6制权，由于正则表达式中的b字符没有做量词修饰，即它表示只占有一个位置的宽度，所以b的控制结束，控制权移交给正则的下一个匹配字符c<br/>
10) 正则表达式控制权在c，匹配位置为6，字符串6位置时候的字符为c，匹配成功，匹配的位置从6变为7制权，由于正则表达式中的c字符没有做量词修饰，即它表示只占有一个位置的宽度，所以c的控制结束，控制权继续向后移交<br/>
11) 正则表达式结束，由于它是全局匹配，控制权再次回到b，但是由于这个时候所匹配的字符串已经结束了，所以整个匹配过程结束

## 4.回溯

当正则匹配不成功的时候，就会尝试进行回溯，回溯成功与否取决于是否有可回溯的位置，若没有会回溯位置，则整个正则表达式匹配失败，控制权交还给表达式的起始位置，正则规则中使用量词修饰，或者使用|的时候，所匹配的位置为可回溯位置（起始第3点介绍控制权的时候，其中几步匹配失败之后，正则尝试进行回溯，但是由于不存在可回溯位置，导致整个表达式匹配失败，控制权移交给整个表达式的起始字符）

依旧是一个简单的例子

```javascript
	let reg = /ab{1,3}bbc/;
	let str = 'abbbc';
```
```
      a      b      b      b      c
  ↑      ↑      ↑      ↑      ↑      ↑
0位置   1位置  2位置   3位置  4位置  5位置
```
依旧按照之前的规则进行匹配
1) 控制权a，起始位置0（整个正则表达式的最开始位置），匹配a成功，匹配位置变为1，a宽度为1，控制权移交给b{1,3}<br/>
2) 控制权b{1,3}，匹配位置1，匹配b成功，匹配位置变为2（可回溯位置），b{1,3}宽度1-3之间，默认为贪婪模式，控制权依旧在b{1,3}<br/>
3) 控制权b{1,3}，匹配位置2，匹配b成功，匹配位置变为3（可回溯回执），b{1,3}宽度1-3之间，默认为贪婪模式，控制权依旧在b{1,3}<br/>
4) 控制权b{1,3}，匹配位置3，匹配b成功，匹配位置变为4，b{1,3}宽度1-3之间，已经达到最大宽度，结束匹配，控制权移交给b<br/>
5) 控制权b(前一个)，匹配位置4，匹配c失败，尝试进行回溯，上一个可回溯位置为3位置，此时b{1,3}这个时候只匹配头两个bb字符，从3位置开始，b(前一个)匹配第三个b字符，回溯成功，b宽度为1，控制权移交给b(后一个)，匹配位置变为4
6) 控制权b(后一个)，匹配位置4，匹配c失败，尝试进行回溯，再上一个可回溯位置为2位置，此时b{1,3}这个时候只匹配b一个字符，控制权移交给b(前一个)，从2位置匹配b(前一个)成功，匹配位置变为3，控制权移交给b（后一个），从3位置匹配b(后一个)成功，匹配位置变为4，控制权移交给c
7) 控制权c，匹配位置4，匹配c成功，匹配位置变为5
8) 字符串结束，整个匹配过程结束

### 回溯总结

正则匹配规则[a, b{1,3}, b, b, c] 分别匹配到了字符[a, b, b, b, c]，整个过程中由于b{1,3}存在可回溯位置，正则默认匹配规则为贪婪模式，b{1,3}首先尽可能多的匹配，直到无法继续匹配的时候将控制权移交给下一个匹配字符，当之后的匹配字符匹配失败的时候，正则表达式尝试从可回溯位置开始进行匹配，如果匹配依旧失败的话，再往前找上一个可回溯位置，直到表达式匹配成功。如果已经没有任何可回溯位置能满足表达式，则整个表达式匹配失败，它将从上次匹配字符串的开始位置的下一个位置再次尝试匹配(例如这次是用/ab{1,3}bbc/匹配'abbbc'，从字符串的0位置匹配整个字符串成功，假如匹配失败，表达式将从字符串1位置开始匹配，也就是从1位置开始，/ab{1,3}bbc/匹配字符'bbbc'，再次匹配失败的时候，表达式从2位置开始重新匹配，也就是用/ab{1,3}bbc/匹配字符串'bbc'以此类推)

## 5.贪婪模式和非贪婪模式

正则默认为贪婪模式，
贪婪模式为尽可能多的匹配，但是非贪婪莫模式不能只解释为尽可能少的匹配
```javascript
let reg = /\w+?/;
let str = 'abcd1234efgh5678';
str.match(reg);
// 结果为['a']
// 这个时候确实可以理解为尽可能少的匹配
reg = /\w+?\d/;
str.match(reg);
// 结果为['abcd1']
// 这个时候如果按照尽可能少的匹配的原则，匹配到的应该是['d1']
// 所以不能单纯的理解为尽可能少的匹配
```

## 6.其他一些匹配回溯的例子
```javascript
let reg = /[a-z]{1,5}1/;
let str = 'abcdef1ghijkl';
```
```
  a   b   c   d   e   f   1   g   h   i   j   k   l
↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑   ↑
0   1   2   3   4   5   6   7   8   9  10   11  12  13
```
### 从0位置开始匹配
[a-z]{1,5}匹配abcde控制权移交给1<br/>
1匹配f失败，尝试回溯，[a-z]{1,5}匹配abcd，控制权交给1<br/>
1匹配e失败，尝试回溯，[a-z]{1,5}匹配abc，控制权交给1<br/>
1匹配d失败，尝试回溯，[a-z]{1,5}匹配ab，控制权交给1<br/>
1匹配c失败，尝试回溯，[a-z]{1,5}匹配a，控制权交给1<br/>
1匹配b失败，[a-z]{1,5}无可回溯位置，[a-z]{1,5}前面的表达式也没有可回溯位置，匹配失败，即从字符串0位置开始匹配失败<br/>

### 从1位置开始匹配
[a-z]{1,5}匹配bcdef控制权移交给1<br/>
1匹配1成功，控制权继续向后移交<br/>
整个表达式结束，并且未声明为全局匹配，整个匹配过程结束<br/>

## 7.回溯例子
```javascript
let str = 'abbbbbc';
let reg = /ab{1,3}b{1,2}bc/;
```
```
/ab{1,3}b{1,2}bc/     abbbbbc<br/>

a                     a
ab{1,3}               ab (a, b{1,3}分别匹配a, b)
ab{1,3}               abb (a, b{1,3}分别匹配a, bb)
ab{1,3}               abbb (a, b{1,3}分别匹配a, bbb)
ab{1,3}b{1,2}         abbbb (a, b{1,3}, b{1,2}分别匹配a, bbb, b)
ab{1,3}b{1,2}         abbbbb (a, b{1,3}, b{1,2}分别匹配a, bbb, bb)
ab{1,3}b{1,2}b        abbbbbc (a, b{1,3}, b{1,2}, b分别匹配a, bbb, bb, c, 匹配失败尝试进行回溯)
ab{1,3}b{1,2}b        abbbbb (a, b{1,3}, b{1,2}, b分别匹配a, bbb, b, b)
ab{1,3}b{1,2}bc       abbbbbc (a, b{1,3}, b{1,2}, b, c分别匹配a, bbb, b, b, c)
```
