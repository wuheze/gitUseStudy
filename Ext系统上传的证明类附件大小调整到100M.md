<!-- ## 任务复盘：系统上传的证明类附件大小调整到100M -->

### 原代码

```js
change: function(thiz, value){
    // 获取组件下面的input的DOM
    var fileDom = thiz.getEl().down('input[type=file]');
    var files = fileDom.dom.files;
    var objDeepCopy = function(source){
        var sourceCopy = {};
        for (var item in source) sourceCopy[item] = source[item];
        return sourceCopy;
    }
    this.allFileList.push(objDeepCopy(fileDom.dom));
    var filesArray = [];
    // 对上传的单个/多个文件进行字段处理
    for(var i = 0; i < files.length; i++){
        var a = {name:files[i].name, lastModified:files[i].name, lastModifiedDate:files[i].lastModifiedDate, size:files[i].size, type:files[i].type, webkitRelativePath:files[i].webkitRelativePath};
        filesArray.push(a);
    }
    // 判断是不是第一次上传文件，如果不是 将上一次的文件合并到这次
    if(me.lastFiles.length > 0){
        for(var j = 0; j < me.lastFiles.length; j++){
            filesArray.push(me.lastFiles[j]);
        }
    }
    me.lastFiles = filesArray;
    files = me.lastFiles;
    var fileArr = [],ck = true,filesize=0;
    // 对本次上传的文件进行单个循环校验
    for(var i = 0; i < files.length; i++){
        var fileName = files[i].name;
        //检查文件类型
        if(!Ext.isEmpty(me.ctrlDocType)){
            var ftype = fileName.substring(fileName.lastIndexOf(".")+1).toLowerCase();
            // 检查数组中是否包含给定元素
            if(!Ext.Array.contains(me.ctrlDocType, ftype)){
                RMsg.error('上传的文件格式不对，只能上传'+me.ctrlDocType.join("，")+'类型的文件!'); 
                ck = false;
                break;
            }
        }
        if (fileName.length > 200){
            RMsg.error("文件名称过长，长度限制为200个字符以内!");
                ck = false;
            break;
        }
        // 将每一次遍历的文件大小加到一起 进行校验
        filesize += files[i].size;
        if (filesize > Dicts.UPLOAD_MAX_SIZE){
            RMsg.error("上传文件大小超过最大值20M！");
            ck = false;
            break;
        }
            fileArr.push(fileName);
    }
    if(ck){
        thiz.setRawValue(fileArr.join('；'));
    }else{
        thiz.setRawValue(fileArr.join('；'));
    }
}			  
```

### 第一次修改

没对change方法进行逐步分析，只根据需求进行单行的代码修改，且测试不认真导致发版后出现问题

```js
Dicts.UPLOAD_MAX_SIZE = 1024 * 1024 * 100

if (files[i].size > Dicts.UPLOAD_MAX_SIZE){
    RMsg.error("上传文件大小超过最大值100M！");
    ck = false;
    ......
}
```

#### 修改后出现的问题

* 用户选择单个文件后，如不符合要求，也没有对lastFiles里面的文件进行清除 导致后续上传都会进行报错，表单中fileArr还会展示本次符合要求的文件名

### 第二次修改

```js

if (files[i].size > Dicts.UPLOAD_MAX_SIZE){
    RMsg.error("上传文件大小超过最大值100M！");
    ck = false;
    // 清除本次上传的文件信息
    this.allFileList.pop()
    Ext.getCmp('file__').reset();
    // 清除数组中该不符合要求的文件信息
    files.pop()
    ......
}
```

<table>
<tr>
<td> 原代码 </td> <td> 更改后 </td>
</tr>
<tr>
<td>

```js
if(me.lastFiles.length > 0){
    for(var j = 0; j < me.lastFiles.length; j++){
    filesArray.push(me.lastFiles[j]);
    }
}
me.lastFiles = filesArray;
```

</td>
<td>

```js
if(me.lastFiles.length > 0){
    me.lastFiles.push(filesArray[0])
}else{
    me.lastFiles = filesArray;
}
```

</td>
</table>

#### 修改后出现的问题

* 清除本次上传的文件信息使用pop方法，没考虑到用户多文件上传时的处理，导致用户选多个文件校验未通过后仍未清理干净数组
* 只删除files，并没有清除lastFiles 二次提交后lastFiles重新给files赋值，导致删除无效

### 第三次修改

<table>
<tr>
<td> 原代码 </td> <td> 更改后 </td>
</tr>
<tr>
<td></td>
<td>

```js
// 声明本次上传文件个数和上次文件上传的个数，以便后续校验有问题将本次上传的所有文件全部清除
var filesLength = files.length
var lastFilesLength = me.lastFiles.length
```

</td>

</tr>
<tr>
<td>

```js
if (files[i].size > Dicts.UPLOAD_MAX_SIZE){
    RMsg.error("上传单个文件大小超过最大值100M！");
    this.allFileList.pop()
    Ext.getCmp('file__').reset();
    files.pop()
    ......
}
```

</td>
<td>

```js
if (files[i].size > Dicts.UPLOAD_MAX_SIZE){
    RMsg.error("上传单个文件大小超过最大值100M！");
    me.lastFiles.splice(lastFilesLength,filesLength)
    this.allFileList.pop()
    Ext.getCmp('file__').reset();
    files.splice(lastFilesLength,filesLength)
    ......
}
```

</td>

</tr>
</table>

### 总结

首先未梳理清原代码逻辑，后需更改未考虑到多种情况，与Ext语法关联不大