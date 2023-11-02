# ext.js的mvc开发模式详解

### 以例子做mvc理解: 

在页面打开的时候加载一个列表，当双击列表中一行数据的时候打开编辑窗口，编辑完成之后点击保存按钮，然后更新列表。

// 图片

* 三个层：Model、View和Controller，这三层是必不可少的
* Ext是全部都用js来写的页面，也就不存在自己写的css和html，js在html中运行是需要一个宿主页面的，那就是一个且仅是唯一的一个index.html文件，在这个html文件中引入所需要的js文件
* js程序需要一个入口，一般是app.js（有的也是Application.js）

##### 第一步：创建入口页面

创建一个html页面，我们使用index.html页面，在这个页面的同一个目录，我们创建一个app的文件夹，在这个文件夹下面用来放置js代码。index.html就是我们的程序宿主页面。

##### 第二步：创建目录结构

在app文件夹下面创建controller、model、store和view文件夹。然后创建app.js作为我们程序的入口文件，并在index.html中引用app.js文件。

##### 第三步：创建model

```Vue
Ext.define('MyApp.model.User', {
    extend: 'Ext.data.Model',
    fields: [
        { name: 'name', type: 'string' },
        { name: 'age', type: 'int' },
        { name: 'phone', type: 'string' }
    ]
});
```

##### 第四步：创建store

虽然store不是mvc中必须的步骤，但是作为数据仓库，store起到了数据存取的作用，grid、form等展现的数据都是通过store来提供的，因此store在extjs mvc开发模式中是必不可少的。

```Vue
Ext.define("MyApp.store.User", {
    extend: "Ext.data.Store",
    model: "MyApp.model.User",
    data: [
        { name: "Tom", age: 5, phone: "123456" },
        { name: "Jerry", age: 3, phone: "654321" }
    ]
});
```

##### 第五步：创建view

为了实现列表和编辑功能，我们需要两个视图，分别是list和edit，那么view的结构如下：

```Vue
Ext.define("MyApp.view.user.List", {
    extend: "Ext.grid.Panel",
    alias: 'widget.userlist',
    store: "User",
    initComponent: function () {
        this.columns = [
            { text: '姓名', dataIndex: 'name' },
            { text: '年龄', dataIndex: 'age', xtype: 'numbercolumn', format: '0'},
            { text: '电话', dataIndex: 'phone' }
        ];
        this.callParent(arguments);
    }
});

----------------------------------------------------------------

Ext.define("MyApp.view.user.Edit", {
    extend: "Ext.window.Window",
    alias: "widget.useredit",
    title: "编辑用户",
    width: 300,
    height: 200,
    layout: "fit",
    items: {
        xtype: "form",
        margin: 5,
        border: false,
        fieldDefaults: {
            labelAlign: 'left',
            labelWidth: 60
        },
        items: [
            { xtype: "textfield", name: "name", fieldLabel: "姓名" },
            { xtype: "numberfield", name: "age", fieldLabel: "年龄" },
            { xtype: "textfield", name: "phone", fieldLabel: "电话" }
        ]
    },
    buttons: [
        { text: "保存", action: "save" }
    ]
});

注意：对于view类的创建，我们需要定义alias，这是为了方便我们通过xtype来创建该组件的实例。（如果没有alias，我们将很难在viewport和controller中使用）
```

##### 第六步：创建controller

controller作为连接model、store和view的桥梁，在mvc开发模式中起到了至关重要的作用。如果说model定义了数据模式、store提供了数据存取的方法、view用来展示数据，那么controller将用来控制具体的数据操作。

```Vue
Ext.define('MyApp.controller.User', {
    extend: 'Ext.app.Controller',
    stores: ['User'],
    models: ['User'],
    views: ['Viewport', 'user.List', 'user.Edit'],
    init: function () {
        this.control({
            'userlist': {
                itemdblclick: this.editUser
            },
            'useredit button[action=save]': {
                click: this.saveUser
            }
        });
    },
    editUser: function (grid, record) {
        var win = Ext.widget("useredit"); 
        win.down("form").loadRecord(record);
        win.show();
    },
    saveUser: function (btn) {
        var win = btn.up("window"),
            form = win.down("form"),
            record = form.getRecord();
        form.updateRecord();
        record.commit();
        win.close();
    }
});

将定义好的model、store、view作为配置项添加到controller中，controller会加载这些文件；
在init方法中为view添加响应事件（这里添加事件的方法是通过query方式获取控件并添加的，这也是为什么要为view添加alias的原因）
editUser方法和saveUser方法则是具体的操作内容
```

##### 第七步：完善app.js文件

```Vue
Ext.application({
    name: "MyApp",
    appFolder: 'app',
    controllers: ["User"],
    autoCreateViewport: true,
    launch: function () {
        // 页面加载完成之后执行

    }
});

name: 应用程序名称
appFolder: 应用程序代码的目录，用来进行动态加载代码的
controllers: 应用程序使用到的控制器
autoCreateViewport: 是否自动创建Viewport，默认为false，我们这里设置为true，当它被设置为true的时候，应用程序会自动创建Viewport，这个Viewport的定义在我们的app/view/viewport.js 中；如果为false的时候，我们需要在launch中手动创建相应的视图。
```

##### 第八步：Viewport.js的定义

Viewport作为我们应用程序的视图面板，可以被单独的定义在一个Viewport.js文件中。它的定义也很简单，通常用来将一个或多个view作为它的子控件。
app/view/viewport.js 代码如下：

```Vue
Ext.define("MyApp.view.Viewport", {
    extend: "Ext.container.Viewport",
    layout: "fit",
    items: {
        xtype:"userlist"
    }
});
```
