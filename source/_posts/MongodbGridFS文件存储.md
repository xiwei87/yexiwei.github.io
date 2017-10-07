---
title: Mongodb GridFS文件存储，Nginx-Gridfs读取需要注意的地方
date: 2012-02-14 17:11:09
tags: [mongo, nosql]
---
#### 一、命令行操作：
```bash
C:Userslwx>mongofiles put D:/www/health.png -h 192.168.20.64 -db test -t png
```
或者
```bash
C:Userslwx>mongofiles put D:/www/health.png -h 192.168.20.64 -db test -t "image/png"
connected to: 192.168.20.64
added file: { _id: ObjectId('4e3a274aad9bf748d735e8d7'), filename: "D:/www/health.png", chunkSize: 262144, uploadDate: new Date(1312433994910), md5: "34cb91d3ebc40c314f4be
e23d57112fd", length: 826796, contentType: "image/png" }
done!
```
contentType:”image/png” 就是-t参数传递的值
<!--more--->
#### 二、用php的mongo扩展写入
```bash
<?php
$db = new Mongo();
$grid = $db->getGridFS();
//将一个文件写入
$grid->storeFile("D:/www/health.png",array('contentType'=>'image/png'));

//将一个上传的文件写入
//hmtl
//<input type="file" name="upload" id="upload" />

$id = $grid->storeUpload('upload',$_FILES['upload']['name']);
$grid->upload(array("_id" => $id),array('$set'=>array('contentType'=>$_FILES['upload']['type'])));
```
当用到nginx-gridfs扩展读取的时候会在相应的文件头中的加入Content-type: image/png’，这样当浏览器访问的时候就能正确解释类型了


