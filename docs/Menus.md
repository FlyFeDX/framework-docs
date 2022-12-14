## [菜单管理]()

### **功能简介**

- 菜单管理可以对系统菜单配置和结构顺序进行统一管理。菜单可以拥有层级结构，一个菜单可以作为另一个菜单的上级,支持灵活调整。

### **逻辑设计**

- 前端界面通过调用安全管理服务,对公告数据实现增删改查等操作.
- bff层根据使用场景,支持直接连接数据库,和第三方接口调用两种方式对数据进行操作.

### **功能及界面**

- 菜单管理界面分为树图结构和子节点详情列表两部分;左侧树图可以逐一展开/折叠,节点支持拖拽调整结构及顺序;
  ![菜单管理](../images/modules/菜单管理.png)
  ![功能角色详情](../images/modules/功能角色-查看.png)

- 右侧列表可对于选中父节点下的子菜单进行增删改查
  ![菜单管理](../images/modules/菜单管理-新增修改.png)
  ![菜单管理](../images/modules/菜单管理-删除.png)

### **接口设计**

- 接口文档:<http://10.10.2.8:9091/project/255/interface/api/cat_1099>
- 界面通过调用service-security服务提供的rest接口完成对菜单信息的操作,具体属性参数在上一章节已做描述.
  ![菜单接口](../images/modules/功能角色&菜单-接口列表.png)

### **性能,限制和约束**

- 点击查询.新增.修改.删除等操作界面呈现数据不得超过2s；
