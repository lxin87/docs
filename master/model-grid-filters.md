# 查询过滤


`model-grid`提供了一系列的方法实现表格数据的查询过滤：

```php
// 禁用
$grid->disableFilter();
// 显示
$grid->showFilter();

// 禁用过滤器按钮
$grid->disableFilterButton();
// 显示过滤器按钮
$grid->showFilterButton();

$grid->filter(function($filter){
    // 展开过滤器
    $filter->expand();
    
    // 在这里添加字段过滤器
    $filter->equal('id', '产品序列号');
    $filter->like('name', 'name');
    ...

});
```

## 过滤器布局

默认布局方式为`rightSide`

### rightSide
```php
use Dcat\Admin\Grid;

$grid->filter(function (Grid\Filter $filter) {
    // 更改为 rightSide 布局
    $grid->rightSide();
    
    ...
});
```
效果

<a href="{{public}}/assets/img/screenshots/filter-right-side.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/filter-right-side.png">
</a>


### panel
```php
use Dcat\Admin\Grid;

$grid->filter(function (Grid\Filter $filter) {
    // 更改为 panel 布局
    $grid->panel();
    
    ...
});
```
效果

<a href="{{public}}/assets/img/screenshots/filter-panel.png" target="_blank">
    <img class="img img-full" src="{{public}}/assets/img/screenshots/filter-panel.png">
</a>


<a name="type"></a>
## 查询类型

目前支持的过滤类型有下面这些:

<a name="equal"></a>
### equal
`sql: ... WHERE `column` = "$input"`：
```php
$filter->equal('column', $label);
```

<a name="nequal"></a>
### notEqual
`sql: ... WHERE `column` != "$input"`：
```php
$filter->notEqual('column', $label);
```
<a name="like"></a>
### like
`sql: ... WHERE `column` LIKE "%$input%"`：
```php
$filter->like('column', $label);
```

### ilike
`sql: ... WHERE `column` ILIKE "%$input%"`：
```php
$filter->ilike('column', $label);
```
<a name="startwith"></a>
### startWith
`sql: ... WHERE `column` LIKE "$input%"`：
```php
$filter->startWith('column', $label);

// 如果需要使用“ilike”
$filter->startWith('column', $label)->ilike();
```

<a name="endwith"></a>
### endWith
`sql: ... WHERE `column` LIKE "%$input"`：
```php
$filter->endWith('column', $label);

// 如果需要使用“ilike”
$filter->endWith('column', $label)->ilike();
```

<a name="gt"></a>
### gt
`sql: ... WHERE `column` > "$input"`：
```php
$filter->gt('column', $label);
```

<a name="lt"></a>
### lt
`sql: ... WHERE `column` < "$input"`：
```php
$filter->lt('column', $label);
```

<a name="ngt"></a>
### ngt
`sql: ... WHERE `column` <= "$input"`：
```php
$filter->ngt('column', $label);
```

<a name="nlt"></a>
### nlt
`sql: ... WHERE `column` >= "$input"`：
```php
$filter->nlt('column', $label);
```

<a name="between"></a>
### between
`sql: ... WHERE `column` BETWEEN "$start" AND "$end"`：
```php
$filter->between('column', $label);

// 设置datetime类型
$filter->between('column', $label)->datetime();

// 设置time类型
$filter->between('column', $label)->time();
```

<a name="in"></a>
### in
`sql: ... WHERE `column` in (...$inputs)`：
```php
$filter->in('column', $label)->multipleSelect(['key' => 'value']);
```

<a name="notIn"></a>
### notIn
`sql: ... WHERE `column` not in (...$inputs)`：
```php
$filter->notIn('column', $label)->multipleSelect(['key' => 'value']);
```

<a name="date"></a>
### date
`sql: ... WHERE DATE(`column`) = "$input"`：
```php
$filter->date('column', $label);
```

<a name="day"></a>
### day
`sql: ... WHERE DAY(`column`) = "$input"`：
```php
$filter->day('column', $label);
```

<a name="month"></a>
### month
`sql: ... WHERE MONTH(`column`) = "$input"`：
```php
$filter->month('column', $label);
```

<a name="year"></a>
### year
`sql: ... WHERE YEAR(`column`) = "$input"`：
```php
$filter->year('column', $label);
```

<a name="where"></a>
### 复杂查询where

可以用where来构建比较复杂的查询过滤

`sql: ... WHERE `title` LIKE "%$input" OR `content` LIKE "%$input"`：
```php
$filter->where('search', function ($query) {

    $query->where('title', 'like', "%{$this->input}%")
        ->orWhere('content', 'like', "%{$this->input}%");

});
```

`sql: ... WHERE `rate` >= 6 AND `created_at` = {$input}`:
```php
$filter->where(function ($query) {

    $query->whereRaw("`rate` >= 6 AND `created_at` = {$this->input}");

}, 'Text');
```

关系查询，查询对应关系`profile`的字段：
```php
$filter->where(function ($query) {

    $query->whereHas('profile', function ($query) {
        $query->where('address', 'like', "%{$this->input}%")->orWhere('email', 'like', "%{$this->input}%");
    });

}, '地址或手机号');
```
<a name="group"></a>
### 过滤器组group
有时候对同一个字段要设置多种筛选方式，可以通过下面的方式实现

```php
$filter->group('rate', function ($group) {
    $group->gt('大于');
    $group->lt('小于');
    $group->nlt('不小于');
    $group->ngt('不大于');
    $group->equal('等于');
});
```
有下面的几个方法可以调用
```php
// 等于
$group->equal();

// 不等于
$group->notEqual();

// 大于
$group->gt();

// 小于
$group->lt();

// 大于等于
$group->nlt();

// 小于等于
$group->ngt();

// 匹配
$group->match();

// 复杂条件
$group->where();

// like查询
$group->like();

// like查询
$group->contains();

// ilike查询
$group->ilike();

// 以输入的内容开头
$group->startWith();

// 以输入的内容结尾
$group->endWith();
```

<a name="scope"></a>
### 范围查询scope
可以把你最常用的查询定义为一个查询范围，它将会出现在筛选按钮的下拉菜单中，下面是几个例子：
```php
$filter->scope('male', '男性')->where('gender', 'm');

// 多条件查询
$filter->scope('new', '最近修改')
    ->whereDate('created_at', date('Y-m-d'))
    ->orWhere('updated_at', date('Y-m-d'));

// 关联关系查询
$filter->scope('address')->whereHas('profile', function ($query) {
    $query->whereNotNull('address');
});

$filter->scope('trashed', '被软删除的数据')->onlyTrashed();
```
scope方法第一个参数为查询的key, 会出现的url参数中，第二个参数是下拉菜单项的label, 如果不填，第一个参数会作为label显示

scope方法可以链式调用任何eloquent查询条件，效果参考Demo

<a name="form"></a>
## 表单类型

<a name="text"></a>
### text

表单类型默认是`text input`，可以设置`placeholder`：

```php
$filter->equal('column')->placeholder('请输入。。。');
```

也可以通过下面的一些方法来限制用户输入格式：

```php
$filter->equal('column')->url();

$filter->equal('column')->email();

$filter->equal('column')->integer();

$filter->equal('column')->ip();

$filter->equal('column')->mac();

$filter->equal('column')->mobile();

// $options 参考 https://github.com/RobinHerbots/Inputmask/blob/4.x/README_numeric.md
$filter->equal('column')->decimal($options = []);

// $options 参考 https://github.com/RobinHerbots/Inputmask/blob/4.x/README_numeric.md
$filter->equal('column')->currency($options = []);

// $options 参考 https://github.com/RobinHerbots/Inputmask/blob/4.x/README_numeric.md
$filter->equal('column')->percentage($options = []);

// $options 参考 https://github.com/RobinHerbots/Inputmask, $icon为input前面的图标
$filter->equal('column')->inputmask($options = [], $icon = 'pencil');
```

<a name="selectResource"></a>
### selectResource
选择数据源，选择弹窗里面的表格数据。

```php
// 单选
$filter->equal('user_id')
    ->selectResource('pages/users')
    ->placeholder('请选择。。。')
    ->options(function ($v) { // options方法用于显示已选中的值
        if (!$v) return $v;
        $userModel = config('admin.database.users_model');

        return $userModel::findOrFail($v)->pluck('name', 'id');
    });

// 多选
$filter->in('user_id')
    ->selectResource('pages/users')
    ->multiple(3) // 最多选择3个选项，不传则不限制
    ->options(function ($v) { // options方法用于显示已选中的值
        if (!$v) return $v;
        $userModel = config('admin.database.users_model');

        return $userModel::findOrFail($v)->pluck('name', 'id');
    });
```

<img style="box-shadow:0 1px 6px 1px rgba(0, 0, 0, 0.12)" width="100%" src="{{public}}/assets/img/screenshots/grid-filter-select-resource.png">


`pages/users`页面实现：
```php
/**
 * Index interface.
 *
 * @return Content
 */
public function index(Content $content)
{
    if (request('_mini')) {
        return $content->body($this->miniGrid());
    }

    ...
}

protected function miniGrid()
{
    $grid = new MiniGrid(new Administrator());
    
    // 指定行选择器选中时显示的值的字段名称
    // 指定行选择器选中时显示的值的字段名称
    // 指定行选择器选中时显示的值的字段名称
    // 如果表格数据中带有 “name”、“title”或“username”字段，则可以不用设置
    $grid->setRowSelectorOptions(['label' => 'username']);

    $grid->id->sortable();
    $grid->username;
    $grid->name;

    $grid->filter(function (Grid\Filter $filter) {
        $filter->equal('id');
        $filter->like('username');
        $filter->like('name');
    });

    return $grid;
}

```

<a name="select"></a>
### select
```php
$filter->equal('column')->select(['key' => 'value'...]);

// 或者从api获取数据，api的格式参考model-form的select组件
$filter->equal('column')->select('api/users');
```

<a name="multipleSelect"></a>
### multipleSelect
一般用来配合`in`和`notIn`两个需要查询数组的查询类型使用，也可以在`where`类型的查询中使用：
```php
$filter->in('column')->multipleSelect(['key' => 'value'...]);

// 或者从api获取数据，api的格式参考model-form的multipleSelect组件
$filter->in('column')->multipleSelect('api/users');
```

<a name="datetime"></a>
### datetime

通过日期时间组件来查询，`$options`的参数和值参考[bootstrap-datetimepicker](http://eonasdan.github.io/bootstrap-datetimepicker/Options/)

```php
$filter->equal('column')->datetime($options);

// `date()` 相当于 `datetime(['format' => 'YYYY-MM-DD'])`
$filter->equal('column')->date();

// `time()` 相当于 `datetime(['format' => 'HH:mm:ss'])`
$filter->equal('column')->time();

// `day()` 相当于 `datetime(['format' => 'DD'])`
$filter->equal('column')->day();

// `month()` 相当于 `datetime(['format' => 'MM'])`
$filter->equal('column')->month();

// `year()` 相当于 `datetime(['format' => 'YYYY'])`
$filter->equal('column')->year();

```

<a name="method"></a>
## 常用方法

<a name="width"></a>
### 设置过滤器宽度
```php
// 设置为“1-12”之间的值，默认值是“3”
$filter->equal('column')->width(3);

// 也可以写死宽度
$filter->equal('column')->width('250px');
```

<a name="default"></a>
### 设置默认值
```php
$filter->equal('column')->default('text');

$filter->equal('column')->select([0 => 'PHP', 1 => 'Java'])->default(1);
```

<a name="expand"></a>
### 展开过滤器
```php
$filter->expand();

$filter->equal('column');
...
```

<a name="withoutInputBorder"></a>
### 不显示过滤器input输入框的边框
```php
$filter->withoutInputBorder();

$filter->equal('column');
...
```

<a name="style"></a>
### 设置过滤器容器样式
```php
$filter->style('padding:0');

$filter->equal('column');
...
```

<a name="padding"></a>
### 设置过滤器容器padding
```php
$filter->padding('10px', '10px', '10px', '10px');

$filter->equal('column');
...
```

<a name="extend"></a>
## 自定义过滤器
下面通过`between`的实现来讲解下怎么自定义过滤器。

首先新建一个过滤器类继承`Dcat\Admin\Grid\Filter\AbstractFilter`：
```php
<?php

namespace Dcat\Admin\Grid\Filter;

use Dcat\Admin\Admin;
use Dcat\Admin\Grid\Filter\Presenter\DateTime;
use Illuminate\Support\Arr;

class Between extends AbstractFilter
{
    // 自定义你的过滤器显示模板
    protected $view = 'admin::filter.between';

    // 这个方法用于生成过滤器字段的唯一id
    // 通过这个唯一id则可以用js代码对其进行操作
    public function formatId($column)
    {
       $id   = str_replace('.', '_', $column);
       $name = $this->parent->getGrid()->getName();

       return ['start' => "{$name}{$id}_start", 'end' => "{$name}{$id}_end"];
    }

    // form表单name属性格式化
    protected function formatName($column)
    {
        $columns = explode('.', $column);

        if (count($columns) == 1) {
            $name = $columns[0];
        } else {
            $name = array_shift($columns);

            foreach ($columns as $column) {
                $name .= "[$column]";
            }
        }

        return ['start' => "{$name}[start]", 'end' => "{$name}[end]"];
    }

    // 创建条件
    // 这里构建的条件支持`Laravel query builder`即可。
    public function condition($inputs)
    {
        if (!Arr::has($inputs, $this->column)) {
            return;
        }

        $this->value = Arr::get($inputs, $this->column);

        $value = array_filter($this->value, function ($val) {
            return $val !== '';
        });

        if (empty($value)) {
            return;
        }

        if (!isset($value['start']) && isset($value['end'])) {
            // 这里返回的数组相当于
            // $query->where($this->column, '<=', $value['end']);
            return $this->buildCondition($this->column, '<=', $value['end']);
        }

        if (!isset($value['end']) && isset($value['start'])) {
            // 这里返回的数组相当于
            // $query->where($this->column, '>=', $value['end']);
            return $this->buildCondition($this->column, '>=', $value['start']);
        }

        $this->query = 'whereBetween';

        // 这里返回的数组相当于
        // $query->whereBetween($this->column, $value['end']);
        return $this->buildCondition($this->column, $this->value);
    }

    // 自定义过滤器表单显示方式
    public function datetime($options = [])
    {
        $this->view = 'admin::filter.betweenDatetime';

        DateTime::collectAssets();

        $this->setupDatetime($options);

        return $this;
    }

    protected function setupDatetime($options = [])
    {
        $options['format'] = Arr::get($options, 'format', 'YYYY-MM-DD HH:mm:ss');
        $options['locale'] = Arr::get($options, 'locale', config('app.locale'));

        $startOptions = json_encode($options);
        $endOptions = json_encode($options + ['useCurrent' => false]);

        // 通过上面格式化后的id对表单进行你想要的操作
        $script = <<<JS
            $('#{$this->id['start']}').datetimepicker($startOptions);
            $('#{$this->id['end']}').datetimepicker($endOptions);
            $("#{$this->id['start']}").on("dp.change", function (e) {
                $('#{$this->id['end']}').data("DateTimePicker").minDate(e.date);
            });
            $("#{$this->id['end']}").on("dp.change", function (e) {
                $('#{$this->id['start']}').data("DateTimePicker").maxDate(e.date);
            });
JS;

        Admin::script($script);
    }
}
```
`admin::filter.between`模板内容如下：
```php
<div class="filter-input col-sm-{{ $width }} "  style="{!! $style !!}">
    <div class="form-group" >
        <div class="input-group input-group-sm">
            <span class="input-group-addon"><b>{!! $label !!}</b></span>
            <input type="text" class="form-control" placeholder="{{$label}}" name="{{$name['start']}}" value="{{ request($name['start'], \Illuminate\Support\Arr::get($value, 'start')) }}">
            <span class="input-group-addon" style="border-left: 0; border-right: 0;">To</span>
            <input type="text" class="form-control" placeholder="{{$label}}" name="{{$name['end']}}" value="{{ request($name['end'], \Illuminate\Support\Arr::get($value, 'end')) }}">
        </div>
    </div>
</div>
```
`admin::filter.betweenDatetime`模板内容如下：
```php
<div class="filter-input col-sm-{{ $width }}"  style="{!! $style !!}">
    <div class="form-group">
        <div class="input-group input-group-sm">
            <span class="input-group-addon"><b>{{$label}}</b>  &nbsp;<i class="fa fa-calendar"></i></span>
            <input type="text" class="form-control" id="{{$id['start']}}" placeholder="{{$label}}" name="{{$name['start']}}" value="{{ request($name['start'], \Illuminate\Support\Arr::get($value, 'start')) }}">
            <span class="input-group-addon" style="border-left: 0; border-right: 0;">To</span>
            <input type="text" class="form-control" id="{{$id['end']}}" placeholder="{{$label}}" name="{{$name['end']}}" value="{{ request($name['end'], \Illuminate\Support\Arr::get($value, 'end')) }}">
        </div>
    </div>
</div>
```

现在只要调用`extend`方法可以使用了，打开`app/Admin/bootstrap.php`，加入以下代码：
```php
Filter::extend('customBetween', Filter\Between::class);
```

使用：
>扩展过滤器方法后IDE默认是不会自动补全的，这时候可以通过`php artisan admin::ide-helper`生成IDE提示文件，具体请参考[IDE自动补全](helpers-ide.md)。

```php
$filter->customBetween('created_at')->datetime();
```
