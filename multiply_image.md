## 多图片支持

zencart原生支持多图片，初接触zencart系统的使用者可能丈二和尚摸不到头脑，因为我们在后台编辑产品看到只有一个提供上传图片/写入图片地址的地方，其实中zencart中实现多图片的功能比想像的还要简单，它是通过图片文件名自动匹配实现的。

控制是否支持多图片，默认开启，进入Catelog -> Product types，选择 Product - General点击edit layout，修改参数Show Product Additional Images = true时，便会显示出多图片。

相当文件夹下，图片文件名规则：imagename***.jpg

如/images/products/image001.jpg，会匹配到/images/products/image001_001.jpg或/images/products/image0010001.jpg等图片文件。