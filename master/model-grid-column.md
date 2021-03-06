# 列的基本使用

### 设置列为可排序
```php
$grid->id()->sortable();
```

### 设置列的宽度
设置列的宽度，当字段内容过长时可以使用这个方法限制列宽度
```php
// px
$grid->long_text->width('300px');
// 百分比
$grid->long_text->width('15%');
```

### 设置表格头HTML属性
设标题的`html`属性
```php
// 修改颜色
$grid->name->setHeaderAttributes(['style' => 'color:#5b69bc']);
```

### 设置列的显示或隐藏
`responsive`方法用于启用[RWD-Table-Patterns](https://github.com/nadangergeo/RWD-Table-Patterns)插件，可以在表格右上角控制显示或隐藏当前字段
```php
$grid->email->responsive();

// 0 不可见
// 1 保持可见，但可以在下拉列表筛选隐藏。
// 2 480px 分辨率以下可见
// 3 640px 以下可见
// 4 800px 以下可见
// 5 960px 以下可见
// 6 1120px 以下可见
$grid->name->responsive(2);
```

#### 默认隐藏当前列
```php
// 隐藏字段
$grid->email->hide();
// 相当于
$grid->name->responsive(0);
```
<a href="{{public}}/assets/img/screenshots/grid-column-responsive.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/grid-column-responsive.png" />
</a>    


### 设置列提示信息
`Grid\Column::help`参数：
 - $help `string` 提示内容
 - $style `string` 提示窗背景颜色，支持“primary”、“success”、“danger”、“purple”
 - $placement `string` 提示窗位置，支持“top”、“left”、“right”、“bottom”

<a href="{{public}}/assets/img/screenshots/grid-column-help.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/grid-column-help.png" />
</a>

```php
$grid->id()->help('提示信息');
```

### 设置列搜索

通过`Grid\Column::filter`方法可以给列设置一个过滤器，可以很方便的根据这一列进行数据表格过滤操作，具体使用方法请参考[列过滤器](model-grid-column-filter.md)。

<a href="{{public}}/assets/img/screenshots/grid-column-filter.png" target="_blank">
    <img class="img" src="{{public}}/assets/img/screenshots/grid-column-filter.png" />
</a>


### 扩展列功能

通过`Grid\Column::macro`方法可以扩展列方法。

在 `app/Admin/bootstrap.php` 中添加以下代码

```php
use Dcat\Admin\Grid;

Grid\Column::macro('myHeader', function ($p1, $p2 = null) {
    // MyHeader 需要实现 Illuminate\Contracts\Support\Renderable 接口
    // 当然这里也可以直接传字符串
    return $this->addHeader(new MyHeader($this, $p1, $p2));
});
```

`MyHeader` 类
```php
use Dcat\Admin\Grid\Column;
use Illuminate\Contracts\Support\Renderable;

class MyHeader implements Renderable
{
    public function __construct(Column $column, $p1, $p2)
    {
        ...
    }
    
    public function render()
    {
        ...
    }
}
```

使用

```php
$grid->user->myHeader($p1, $p2);

$grid->first_name->myHeader($p1);
```


