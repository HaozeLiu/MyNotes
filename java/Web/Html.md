# HTML

HTML的好处是我们不需要使用其他的环境和服务
只需要创建一个`.html`文件,然后使用浏览器打开就可以了

首页`html`文件一般是`index.html`,如果你部署了`www.xxx.com`,根域名将显示`index.html`中的内容



## HTML标签&页面结构

`<tagname>content</tagname>`

e.g. : `<h1>一些内容</h1>  <p>一些内容</p>`

若标签中无内容  `<tagname></tagname>`
				->自闭合标签: `<tagname />`
				->`<tagname>`(HTML5中这样写就可以)

结构:

- 标题标签:`<h1>head one</h1>  <h2>head two</h2>...`

- 段落标签:`<p>content</p>`

- 行级元素 块级元素:

- - 加粗`<strong>content</strong>`
  - 斜体`<em>content</em>`
  - 链接`<a href="https://baidu.com">content</a>` 如果添加`target="_blank"`就会新建页面打开网址而不是默认的当前页面打开

- 列表标签

- - 无序列表`<ul> <li>列表项1</li><li>列表项2</li></ul> `  这个类似于markdown里的 "-" + 空格

- - 有序列表:无序列表里的`<ul>`换成`<ol>`即可

- **表格**:`<table></table>`之间的就是表格

- - `<thead></thead>`之间定义的是表格的标题部分 - `<tbody></tbody>`之间定义的是表格的主体部分

- - `<tr></tr>`之间的为一行(`tr即 table row`)

  - `<th></th>`中的内容为表头单元格(`th即为 table header`)

  - `<td></td>`中的内容为数据单元格(`td即为 table data`)

    e.g.:

    ```html
    <table>
               <thead>
                <tr>
                    <th>姓名</th>
                    <th>邮箱</th>
                    <th>年龄</th>
                </tr>
               </thead> 
               <tbody>
                <tr>
                    <td>LiuHaoze</td>
                    <td>Haozee@foxmail.com</td>
                    <td>19</td>
                </tr>
                <tr>
                    <td>LiuHaoze</td>
                    <td>Haozee@foxmail.com</td>
                    <td>19</td>
                </tr>
                <tr>
                    <td>LiuHaoze</td>
                    <td>Haozee@foxmail.com</td>
                    <td>19</td>
                </tr>
               </tbody>
            </table>
    ```

    呈现的效果为:

    | 姓名     | 邮箱               | 年龄 |
    | -------- | ------------------ | ---- |
    | LiuHaoze | Haozee@foxmail.com | 19   |
    | LiuHaoze | Haozee@foxmail.com | 19   |
    | LiuHaoze | Haozee@foxmail.com | 19   |

    
    

  - 表单:`<form></form>`之间的内容

  - `<div></div>`之间的可以独占一行.

    - `<lable></lable>`间填标题,类似于表单的标题
    - `<input type = "xxx">`输入的类型,可以填txt
    - 特殊的,可以不用`input`而用`<textatea></textarea>`包裹较长的输入内容
    - `<select></select>`之间代表选择(不是撰写填表而是做选择填表)
    - - select之中`<option value="xxx">x<option>`代表选项

  - 按钮:`<button>点我</button>`

  - 图片:`<img src="图片链接" alt="图片出问题没显示时现实的文字">`

```html
<!DOCTYPE html>
<html> 
    <head> 
        <title>html tutorial</title>
    </head>
    <body>
        <!-- 标题标签 h -->
        <h1>Heading One</h1>
        <h2>Heading Two</h2>
        <h3>Heading Three</h3>
        <h4>Heading Four</h4>
        <h5>Heading Five</h5>
        <h6>Heading Six</h6>

        <!-- 段落标签 -->
        <p>
            <strong>This is strong thing</strong> ipsum dolor sit amet consectetur, 
            <em>This is emphasis thing</em> elit. Repellat ea, 
            <a href="https://baidu.com">This is anchor</a> 
            <a href="https://baidu.com" target="_blank">If you add "_blank" it creates a new page.</a> veritatis officiis sed iure libero beatae corporis. Nihil id officiis consequuntur vero vitae reiciendis repudiandae, ut repellendus dolor?
        </p>
        <p>
            Lorem ipsum dolor sit amet consectetur, adipisicing elit. Repellat ea, atque aperiam veritatis officiis sed iure libero beatae corporis. Nihil id officiis consequuntur vero vitae reiciendis repudiandae, ut repellendus dolor?
        </p>
        <!-- 列表标签: 无序列表 有序列表 -->
        <!-- 无序列表 -->
        <ul>
            <li>列表项1</li>
            <li>列表项2</li>
            <li>列表项3</li>
            <li>列表项4</li>
        </ul>
        <!-- 有序列表 -->
        <ol>
            <li>列表项1</li>
            <li>列表项2</li>
            <li>列表项3</li>
            <li>列表项4</li>
        </ol>
        <!-- 表格标签 -->
        <table>
           <thead>
            <tr>
                <th>姓名</th>
                <th>邮箱</th>
                <th>年龄</th>
            </tr>
           </thead> 
           <tbody>
            <tr>
                <td>LiuHaoze</td>
                <td>Haozee@foxmail.com</td>
                <td>19</td>
            </tr>
            <tr>
                <td>LiuHaoze</td>
                <td>Haozee@foxmail.com</td>
                <td>19</td>
            </tr>
            <tr>
                <td>LiuHaoze</td>
                <td>Haozee@foxmail.com</td>
                <td>19</td>
            </tr>
           </tbody>
        </table>
        <br>
        <hr>
        
        <!-- 表单 form -->
        <form>
            <div>
                <label>姓名:</label> 
                <input type="text">
            </div>
            <br />
            <div>
                <label>年龄: </label>
                <input type="text">
            </div>
            <div>
                <label>信息</label>
                <textarea></textarea>
            </div>
            <div>
                <select>
                    <option value="male">男</option>
                    <option value="female">女</option>
                    <option value="other">其他</option>
                </select>
            </div>
            
            <!-- 按钮 button -->
        <button>点我</button>

        <br>

        <!-- 图片 img -->
        <img src="图片链接" alt="出问题没显示出来图片时显示的文字">
            
        </form>
    </body>
</html>
```

