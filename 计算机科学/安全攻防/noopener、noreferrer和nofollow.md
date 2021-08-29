## noopener和noreferrer漏洞机制
通过链接打开的浏览器新窗口，可通过`window.opener`获取源窗口的window，并操作：
``` javascript
window.opener.location.href ="黑客.com"
```
也可通过`document.referrer`获取源页面的URL，进行不良统计分析；

## 预防
### 预防
在标签上增加`rel="noopener"` 把`window.opener`设置为null；

### 预防referrer
- 在标签上增加`rel="noreferrer"`阻止获取来源referrer；
- 页面reload一次，刷掉referrer
- 设置页面meta：`<meta content="never" name="referrer">`


## nofollow
搜索引擎会对引用很多的链接增加搜索权重，博客上的友链也是利用了这一点；但作为一个内容生产平台就要注意了，你肯定不希望用户借用你的SEO在内容区创造大量自己的「不良」链接，知乎、简书、CSDN都对此作了防范：
```
<a href="https://link.zhihu.com/?target=https%3A//www.目标地址.com/" target="_blank" rel="nofollow noreferrer" ></a>
```
统一先跳到link.zhihu.com，目标链接作为参数，并强提醒用户即将离开，注意安全；并增加了属性`rel="nofollow noreferrer"`阻止了源信息；nofollow告诉搜索引擎，此链接不要增加统计排名；

作为内容平台这么做的好处有：
- 防止不良SEO被利用
- 通过中间页link自己更有把控权，并能对挑出的数据做分析
- 免责声明

>[[计算机安全]] [[document.referrer]] [[noopener]] [[、noreferrer]] [[nofollow]]