# 表单回调

`Form`目前提供了下面几个方法来接收回调函数：

### creating

在新增页面调用（非提交操作）

```php
$form->creating(function (Form $form) {
    if (...) { // 验证逻辑
		$form->responseValidationMessages('title', 'title格式错误');
		
		// 如有多个错误信息，第二个参数可以传数组
		$form->responseValidationMessages('content', ['content格式错误', 'content不能为空']);
	}
});
```

### editing

在编辑页面调用（非提交操作）

```php
$form->editing(function (Form $form) {
    if (...) { // 验证逻辑
		$form->responseValidationMessages('title', 'title格式错误');
		
		// 如有多个错误信息，第二个参数可以传数组
		$form->responseValidationMessages('content', ['content格式错误', 'content不能为空']);
	}
});
```

### submitted

在表单提交前调用

```php
$form->submitted(function (Form $form) {
    // 获取用户提交参数
    $title = $form->title;
    
    // 上面写法等同于
    $title = $form->input('title');
});
```

### saving
保存前回调

```php
$form->saving(function (Form $form) {
    // 判断是否是新增操作
    if ($form->isCreating()) {
    
    }
});
```

### saved

保存后回调，此事件新增和修改操作共用，通过第二个参数`$result`可以判断数据是否保存成功。

> {tip} 新增页面下，`$result`的值是新增记录的自增ID

```php
$form->saved(function (Form $form, $result) {
    // 判断是否是新增操作
    if ($form->isCreating()) {
    	// 自增ID
    	$newId = $result;
    	// 也可以这样获取自增ID
    	$newId = $form->getKey();
    	
    	if (! $newId) {
    		return $form->error('数据保存失败');
    	}
    	
    	return;
    }
    
    // 修改操作
});
```
> {tip} `$form->repository()->eloquent()`為當前新增或編輯後的eloquent

```php
$form->saved(function (Form $form, $result) {
    // 在表單保存後獲取eloquent
    $form->repository()->eloquent()->update(['data' => 'new']);
});
```

### deleting

删除前回调

```php
$form->deleting(function (Form $form, $result) {
	// 获取待删除行数据，这里获取的是一个二维数组
	$data = $form->model()->toArray();
	
	// 通过 $result 可以判断数据是否删除成功
	if (! $result) {
		return $this->error('数据删除失败');
	}

    // 跳转
    return $form->redirect('auth/user', [
        'message' => '操作成功',
    ]);
    
});
```

### deleted

删除后回调，通过第二个参数`$result`可以判断数据是否删除成功。

```php
$form->deleted(function (Form $form, $result) {
    // 获取待删除行数据，这里获取的是一个二维数组
	$data = $form->model()->toArray();
});
```

### uploading

图片、文件上传事件

> {tip} 文件上传是一个独立的api请求，这个事件内`redirect`方法是无效的。

```php
use Dcat\Admin\Form;
use Dcat\Admin\Form\Field;
use Dcat\Admin\Contracts\UploadField as UploadFieldInterface;
use Symfony\Component\HttpFoundation\File\UploadedFile;

$form->uploading(function (Form $form, UploadFieldInterface $field, UploadedFile $file) {
	// $file 即是当前上传的完整文件
	
	/* @var Field $field */
	// 获取文件上传字段名称
	$column = $field->column();
});
```

### uploaded

图片、文件上传完毕事件

> {tip} 文件上传是一个独立的api请求，这个事件内`redirect`方法是无效的。

```php
use Dcat\Admin\Form;
use Dcat\Admin\Form\Field;
use Dcat\Admin\Contracts\UploadField as UploadFieldInterface;
use Symfony\Component\HttpFoundation\File\UploadedFile;

$form->uploaded(function (Form $form, UploadFieldInterface $field, UploadedFile $file, $response) {
	// $file 即是当前上传的完整文件
	
	/* @var Field $field */
	// 获取文件上传字段名称
	$column = $field->column();
	
	$response = (array) $response->getData();
	
	// 文件上传成功
	if ($response['status']) {
		// 文件访问地址
		$url = $response['url'];
	}
});
```

### 获取模型中的数据
```php
$form->saved(function (Form $form) {

    $id = $form->getKey();

    $username = $form->model()->username;
    
    // 获取最终保存的数组
    $updates = $form->updates();
});
```

### 修改模型中的数据
修改模型中的数据需要配合隐藏表单使用。举例：
```php
$form->hidden('author_id');

$form->saving(function (Form $form) {

    $form->author_id = 1;
});
```

### 页面跳转

> {tip} 此方法在`creating`、`editing`、`uploading`、`uploaded`事件中均不可用。

```php
// 跳转并提示成功信息
$form->saved(function (Form $form) {
    return $form->redirect('auth/user', '保存成功');
});

// 跳转并提示错误信息
$form->saved(function (Form $form) {
    return $form->redirect('auth/user', [
        'message' => '系统错误',
        'status' => false,
    ]);
});
```

### 仅返回错误信息但不跳转

通过`error`方法可以在提交表单时返回错误提示信息

> {tip} 此方法在`creating`、`editing`、`uploading`、`uploaded`事件中均不可用。

```php
$form->saving(function (Form $form) {
    return $form->error('系统异常');
});
```

### 返回字段验证出错信息

通过`responseValidationMessages`方法可以很方便的返回字段验证出错信息，而不需要使用`Laravel validation`功能。

普通使用
```php
protected function form()
{
	return Form::make(new Model(), function (Form $form) {
		if (...) { // 验证逻辑
			$form->responseValidationMessages('title', 'title格式错误');
			
			// 如有多个错误信息，第二个参数可以传数组
			$form->responseValidationMessages('content', ['content格式错误', 'content不能为空']);
		}
	});
}
```

在事件中使用
> {tip} 此方法仅在`submitted`事件中可用

```php
$form->submitted(function (Form $form) {
	// 接收表单参数
	$title = $form->title;

    if (...) { // 验证逻辑
        $form->responseValidationMessages('title', 'title格式错误');
        
        // 如有多个错误信息，第二个参数可以传数组
        $form->responseValidationMessages('content', ['content格式错误', 'content不能为空']);
    }
});
```
