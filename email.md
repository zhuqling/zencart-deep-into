# 邮件

邮件的类型包括：

1. 新注册邮件、欢迎

    由includes/modules/create_account.php里调用，使用email_template_welcome.html模板
    if (trim(EMAIL_SUBJECT) != 'n/a') 决定是否发送邮件

    会发送welcome_extra邮件

2. 忘记密码

    由pages/password_forgotten/header_php.php文件里调用，使用email_template_password_forgotten.html邮件模板

    始终发送

3. 新订单

    通过order类的function send_order_email($zf_insert_id, $zf_mode)方法实现，传递给zen_mail的参数module为checkout_extra，使用email_template_checkout.html模板文件，

    始终发送

    会发送checkout_extra邮件

4. 订单状态变更

    在admin/orders.php里，调用email_template_order_status.html邮件模板
    
    当notify参数为1时发送

5. 联系我们

    在pages/contact_us/header_php.php里实现，调用email_template_contact_us.html邮件模板文件

    始终发送

6. Coupon

    由admin/coupon_admin.php调用，使用email_template_coupon.html模板

    始终发送

    会发送coupon_extra邮件

7. GV mail

    在admin/gv_mail.php里调用，使用email_template_gv_mail.html模板

    始终发送

    会发送gv_mail_extra邮件

8. GV queue

    在admin/gv_queue.php里调用 ，使用email_template_gv_queue.html模板

    始终发送

    会发送gv_queue_extra邮件

9. GV send

    在pages/gv_send/header_php.php，使用email_template_gv_send.html模板

    始终发送

    会发送gv_send_extra邮件

10. 直接邮件

    在admin/mail.php，使用email_template_direct_email.html模板，直接发送邮件给指定的顾客

11. 低库存

    在生成新订单时，order类的function send_order_email($zf_insert_id, $zf_mode)方法同时发磅低库存提醒邮件，传递给zen_mail的参数module为low_stock，使用email_template_low_stock.html邮件模板文件

    当后台设置需发提醒并添加了收件地址时发送

12. 订阅邮件

    在admin/includes/modules/newsletter/newsletter.php，调用 email_template_newsletters.html模板

13. 产品提醒

    在admin/includes/modules/newsletter/product_notification.php里，调用  email_template_product_notification.html模板


14. 告诉朋友

    email_template_tell_a_friend.html，此功能已取消

15. 用户授权

    在admin/customers.php里，使用email_template_default.html模板

16. 留评价

    pages/product_reviews_write/header_php.php，使用reviews_extra作为module参数，没有email_template_reviews_extra.html模板存在，使用email_template_default.html模板
