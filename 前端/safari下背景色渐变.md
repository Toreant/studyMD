在safari下，使用linear-gradient时，会莫名出现灰色的背景。查阅资料后发现，原来，Safari在渐变中对颜色的处理是同时渐变rgba()中的每一个参数，因此该例子中实际上是在渐变从黑色到颜色，从全透明到不透明。因此，中间出现了灰色。 

#### 解决方案  

在渐变的结果处，使用开始的颜色，并改变透明度

```css
/*原本的样式*/
background-image: linear-gradient(0deg,rgba(0,196,189,.06),#fff);

/*改进后的样式*/
background-image: linear-gradient(0deg,rgba(0,196,189,.06),rgba(0,196,189,0));
```
