# 错误/消息提醒

前面我们已经介绍过，在页面处理代码里可以设置消息堆栈。

```php
$messageStack->add('account_password', ENTRY_PASSWORD_NEW_ERROR);
```

消息的显示，是在模板里实现的，消息的显示与常规消息类型见如下代码。

1. Header类型消息和error_message以及info_message
头部tpl_header.php

```php
if ($messageStack->size('header') > 0) {
    echo $messageStack->output('header');
  }
  if (isset($_GET['error_message']) && zen_not_null($_GET['error_message'])) {
  echo htmlspecialchars(urldecode($_GET['error_message']));
  }
  if (isset($_GET['info_message']) && zen_not_null($_GET['info_message'])) {
   echo htmlspecialchars($_GET['info_message']);
```

其中header类型的消息将在网页最上方显示。同时$_GET['error_message']和$_GET['info_message']如被指定值时也将显示，$_GET['error_message']定位为错误类型，$_GET['info_message']定位为通知类型。

2. Upload类型
主页面tpl_main_page.php，页面内容上方。

```php
<!-- bof upload alerts -->
<?php if ($messageStack->size('upload') > 0) echo $messageStack->output('upload'); ?>
<!-- eof upload alerts -->
```

upload类型定位为上传错误类型。

3. 自定义

```php
<?php if ($messageStack->size('addressbook') > 0) echo $messageStack->output('addressbook'); ?> 
```

用户片定义的类型，由用户自己命名，只要在页面代码里的类型名称与模板输出时一致即可。
