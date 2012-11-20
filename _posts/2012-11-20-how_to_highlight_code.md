## 折磨

其实也算不上折磨啦,不过对于不了解html/css/js的人,想要一下子搞定还真麻烦.

## 几个步骤

要知道以下几点:

pages从本质来说是替换模版,把定义好的东西替换成`{}`或者`{%%}`这样类似的东西

对于代码来说,替换之后其实就是在代码外围加了两个标签而已,比如


    int main()
        {
        printf("hello world\n");
        return 0;
    }

在pages的模版引擎替换之后,就是如下代码:

    <pre>
    <code>
    int main()
    {
        printf("hello world\n");
        return 0;
    }
    </code>
    </pre>


这些节点元素(即`node`)是可以通过js动态的添加属性的,比如在[prettify](https://code.google.com/p/google-code-prettify/)上面说的,需要在`pre`标签添加`prettyprint`的`class`属性才行,这样我们通过简单的js来实现

    var pres = document.getElementsByTagName("pre");
    for (var i = 0; i < pres.length; i++)
    {
        pres[i].setAttribute("class", "prettyprint");
    }
    
这样就把所有的`pre`标签添加了

整个prettify需要提前加载进来,按照[readme](https://google-code-prettify.googlecode.com/svn/trunk/README.html)的说法,需要添加`onload`

    var bodies = document.getElementsByTagName("body");
    bodies[0].setAttribute("onload", "prettyPrint()");

## 总结

其实通过以上四步就简单解决了代码高亮的问题,不过这还只是基本的问题,搭建一个技术类
的博客还需要很多知识,这个还要慢慢的学习.

希望能有所帮助.
