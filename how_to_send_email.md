## 发送邮件的过程
	
发送邮件的任务实际是交给functions/functions_email.php中的 `zen_mail($to_name, $to_address, $email_subject, $email_text, $from_email_name, $from_email_address, $block=array(), $module='default', $attachments_list='' ) ` 方法执行的

Zen_mail再根据调用时的参数决定是使用文本构造方式还是使用模块，模块的提取是由 `function zen_build_html_email_from_template($module='default', $content='')` 方法实现的，方法会自动调用相应的邮件模板并将参数替换。

邮件模板位于email目录，所有文件以email_template_开头，以.html为后缀

查找模板文件的方法为，由指定的模板名称以email_template_<module>.html规则的文件名在email文件里查找，当以指定的模板名称找不到，会自动将模板名称里的extra或admin删除后，重新查找

Email目录支持分语言，在email文件夹下新建各语言代码的目录，使会自动查找相应的邮件模板，英语的在email根目录。

模板文件夹的查找顺序为：

1. `Emails/email_template_<current_page_base>.html`
2. $block['EMAIL_TEMPLATE_FILENAME'].html
3. `Emails/email_template_<module>.html` 注：module已经将其中的_extra/_admin删除
4. Emails/email_template_default.html

## 调试邮件模板

在functions_email.php，将 `if (!defined('EMAIL_SYSTEM_DEBUG')) define('EMAIL_SYSTEM_DEBUG','0');` 里的EMAIL_SYSTEM_DEBUG常量设置为preview即可。

决定是否发送邮件的条件：

- 后台
- 其它对应功能