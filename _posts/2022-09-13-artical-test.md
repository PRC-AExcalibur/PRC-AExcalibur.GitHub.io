---
title:  博客中文测试
categories:
- Daily 
tags:
- daily 
---

测试中文博客。

---

# Heading 1

## Heading 2

### Heading 3

#### Heading 4

##### Heading 5

###### Heading 6

---

## Paragraph

Lorem ipsum dolor sit amet, [test link]() consectetur adipiscing elit. **Strong text** pellentesque ligula commodo viverra vehicula. *Italic text* at ullamcorper enim. Morbi a euismod nibh. <u>Underline text</u> non elit nisl. ~~Deleted text~~ tristique, sem id condimentum tempus, metus lectus venenatis mauris, sit amet semper lorem felis a eros. Fusce egestas nibh at sagittis auctor. Sed ultricies ac arcu quis molestie. Donec dapibus nunc in nibh egestas, vitae volutpat sem iaculis. Curabitur sem tellus, elementum nec quam id, fermentum laoreet mi. Ut mollis ullamcorper turpis, vitae facilisis velit ultricies sit amet. Etiam laoreet dui odio, id tempus justo tincidunt id. Phasellus scelerisque nunc sed nunc ultricies accumsan.

Interdum et malesuada fames ac ante ipsum primis in faucibus. `Sed erat diam`, blandit eget felis aliquam, rhoncus varius urna. Donec tellus sapien, sodales eget ante vitae, feugiat ullamcorper urna. Praesent auctor dui vitae dapibus eleifend. Proin viverra mollis neque, ut ullamcorper elit posuere eget.

> Praesent diam elit, interdum ut pulvinar placerat, imperdiet at magna.

Maecenas ornare arcu at mi suscipit, non molestie tortor ultrices. Aenean convallis, diam et congue ultricies, erat magna tincidunt orci, pulvinar posuere mi sapien ac magna. Vestibulum ante ipsum primis in faucibus orci luctus et ultrices posuere cubilia Curae; Praesent vitae placerat mauris. Nullam laoreet ante posuere tortor blandit auctor. Sed id ligula volutpat leo consequat placerat. Mauris fermentum dolor sed augue malesuada sollicitudin. Vivamus ultrices nunc felis, quis viverra orci eleifend ut. Donec et quam id urna cursus posuere. Donec elementum scelerisque laoreet.

## List Types

### Definition List (dl)

<dl><dt>Definition List Title</dt><dd>This is a definition list division.</dd></dl>

### Ordered List (ol)

1. List Item 1
2. List Item 2
3. List Item 3

### Unordered List (ul)

- List Item 1
- List Item 2
- List Item 3

## Table

| Table Header 1 | Table Header 2 | Table Header 3 |
| --- | --- | --- |
| Division 1 | Division 2 | Division 3 |
| Division 1 | Division 2 | Division 3 |
| Division 1 | Division 2 | Division 3 |

## Misc Stuff - abbr, acronym, sub, sup, etc.

Lorem <sup>superscript</sup> dolor <sub>subscript</sub> amet, consectetuer adipiscing elit. Nullam dignissim convallis est. Quisque aliquam. <cite>cite</cite>. Nunc iaculis suscipit dui. Nam sit amet sem. Aliquam libero nisi, imperdiet at, tincidunt nec, gravida vehicula, nisl. Praesent mattis, massa quis luctus fermentum, turpis mi volutpat justo, eu volutpat enim diam eget metus. Maecenas ornare tortor. Donec sed tellus eget sapien fringilla nonummy. <acronym title="National Basketball Association">NBA</acronym> Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus.  <abbr title="Avenue">AVE</abbr>


This is a link post. Clicking on the link should open [baidu](http://www.baidu.com/) in a new tab or window.

This post is used for testing tag plugins. See [docs](http://zespia.tw/hexo/docs/tag-plugins.html) for more info.



# Block Test

### Normal blockquote

> Praesent diam elit, interdum ut pulvinar placerat, imperdiet at magna.

## Code Block

### Inline code block

This is a inline code block: `python`, `print 'helloworld'`.

### Normal code block

```
alert('Hello World!');

```
print "Hello world"


### Highlight code block

```javascript
alert( 'Hello, world!' );
```

```python
print "Hello world"
```

```c++
#include <iostream>
using namespace std;

void foo(int arg1, int arg2)
{

}

int main()
{
  string str;
  foo(1, 2);
  cout << "Hello World" << endl;
  return 0;
}
```



# MathJax Test

$$a^2 + b^2 = c^2$$

{% highlight r %}
$$a^2 + b^2 = c^2$$
{% endhighlight %}

To display inline math use ```\\( ... \\)``` like this ```\\( sin(x^2) \\)``` which gets
rendered as \\( sin(x^2) \\)


Here's another example using type ```\mathsf```

{% highlight r %}
$$ \mathsf{Data = PCs} \times \mathsf{Loadings} $$
{% endhighlight %}

which gets displayed as

$$ \mathsf{Data = PCs} \times \mathsf{Loadings} $$

Or even better:

{% highlight r %}
\\[ \mathbf{X} = \mathbf{Z} \mathbf{P^\mathsf{T}} \\]
{% endhighlight %}

is displayed as

\\[ \mathbf{X} = \mathbf{Z} \mathbf{P^\mathsf{T}} \\]

If you want to use subscripts like this \\( \mathbf{X}\_{n,p} \\) you need to scape the
underscores with a backslash like so ``` \mathbf{X}\_{n,p} ```:

{% highlight r %}
$$ \mathbf{X}\_{n,p} = \mathbf{A}\_{n,k} \mathbf{B}\_{k,p} $$
{% endhighlight %}



will be displayed as

\\[ \mathbf{X}\_{n,p} = \mathbf{A}\_{n,k} \mathbf{B}\_{k,p} \\]
