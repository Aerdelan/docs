## html基础

#### 无序列表（使用很多）

注意：ul里面只能放li,  li里面可以放任何内容。

#### 有序列表（使用很少）

注意：ol里面只能放li,  li里面可以放任何内容。

#### 自定义列表（使用较少）

注意：dl里面只能放dt+dd   ;  dd里面可以放任何内容。

#### 表格table

表格作用  --- 收集/展示  数据用的。

table里面只能放tr,tr里面放td,td里面可以放任何内容。

#### 表格属性（

```html
border    表格的边框大小
width     表格的宽度
height    表格的高度
 cellspacing    设置单元格与单元格之间距离（了解）
 cellpadding    设置单元格内容到边框的距离 （了解）
 <table border="1" width="249" align="center" cellspacing="0" cellpadding="20">
        <tr>
            <td>姓名</td>
            <td>性别</td>
            <td>年龄</td>
        </tr>
      ......
    </table>
```

#### 表格标题caption

注意：写在table标签里面，但是展示在table表格 的上面；并且居中显示。

#### 表头单元格th

特点：加粗 居中 显示，一般都用于表格的第一行。

用法：只需要把该tr里面的td换成 th 即可！

#### 表格的解构标签

```html
划分表格区域用的，没有实际大的意义
thead   tbody    tfoot
```

#### 合并单元格

步骤：

1.先分析是跨行(rowspan)还是跨列(colspan)

2.找到合并单元格的位置，书写合并属性(rowspan/colspan)。

3.删除多余的单元格

### 表单-作用 收集信息和数据

#### input控件

##### type属性：的值不同，展示出来的效果不一样，如下：

type="text"	文本框

type="password"	密码框

type="radio"	单选按钮

type="checkbox"	复选框

type="file"	上传控件

type="submit"   提交按钮

type="reset"   重置按钮    ---》注意：要想实现内容重置，控件必须被form表单域 包含！！

type="button"    普通按钮   --》注意：后期主要搭配javascript使用！！

##### input 单选按钮或者 复选按钮 的默认选中用checked

```html
 性别：<input type="radio" name="sex" checked> man
    <input type="radio" name="sex"> woman 
    爱好：<input type="checkbox">唱歌 <input type="checkbox">旅游<input type="checkbox" checked>学习
```



##### placeholder属性

作用 -- 提高用户体验度

##### name属性

作用 -- 表示单签表单控件的名字(含义)

##### value属性

作用  -- (后端只认前端表单里面的value值)最终把用户填写的value值，传送给后端收集。

##### multiple属性

作用  -- 可以选择多文件上传

```html
   <!-- action代表要提交给谁（地址）
    method 提交方式 值 get post  暂时了解，后面详细讲解 -->
    <form action="index.php">
        姓名：<input type="text" name="user" value="姓名">
        <br><br> 密码：
        <input type="password" value="" name="pwd">
        <br><br> 昵称： <input type="text" value="" name="nking">
        <!-- <input type="submit"> -->
        <!-- <input type="reset" value="重置"> -->
        <!-- <input type="button" value="普通按钮"> -->
        <!-- 新增的按钮标签button -- 也能实现提交功能 -->
        <button>按钮</button>
    </form>
```

#### select下拉选框

**注意：下拉选框的默认选中用  selected**  

```html
 <select>
        <option>济南</option>
        <option selected>青岛</option>
        <option>日照</option>
    </select>
```

#### 文本域标签

```html
 <!-- 注意：文本域的默认提示文本，直接写在标签中间
    文本域的属性 cols rows，几乎不用，忘记也没关系！
    -->
    <textarea cols="30" rows="4">请留言</textarea>
```

#### label标签

作用  -- 增加用户体验度。

```html
<!-- 方法一： -->
    性别 -- <input type="radio" name="sex" id="nan"> <label for="nan">男</label>
    <br><br>
    <!-- 方法二： -->
    <label>
        <input type="radio" name="sex"> 女
    </label>
```

#### 没有语义的div和span

```html
 <!-- div和span是没有语义的，后期搭配css布局来用 -->
    <!-- 今天知道：div独占一行  span一排可以放多个 -->
    <div>我是div大盒子，独占一行</div>
    <div>我是div大盒子，独占一行</div>

    <span>我是span小盒子</span>
    <span>我是span小盒子</span>
    <span>我是span小盒子</span>
```

#### html5新增的语义化标签

```html
<!-- 
    在手机端，推荐大量使用有语义化的标签
    -->
    <header>网页的头部布局</header>
    <nav>网页导航</nav>
    <footer>网页底部布局</footer>
    <aside>侧边栏</aside>
    <section>网页区块</section>
    <article>文章</article>
    <main>网页的主体内容</main>
```

#### 实体字符

```html
大于号和  小于号   &gt;   &lt;
版权  &copy;
钱  &yen;
空格  &nbsp;
```

#### 视频video

```html
 <!-- 谷歌浏览器把自动播放禁用了。如果想实现自动播放，需要添加静音属性muted,
    但是：自动实现了，视频没声音了 -->
    <video src="./video.mp4" controls autoplay muted loop></video>
```