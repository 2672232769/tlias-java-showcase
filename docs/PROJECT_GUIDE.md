# 从部门查询理解项目

示例：在部门管理页面查看全部部门。

| 层次 | 代表文件 | 作用 |
| --- | --- | --- |
| 页面 | `frontend/src/views/dept/index.vue` | 显示列表、接收操作 |
| 请求 | `frontend/src/api/dept.js` | 调用部门 API |
| Controller | `backend/src/main/java/com/itheima/controller/DeptController.java` | 接收 HTTP 请求、返回统一结果 |
| Service | `backend/src/main/java/com/itheima/service/impl/DeptServiceImpl.java` | 组织业务处理 |
| Mapper | `backend/src/main/java/com/itheima/mapper/DeptMapper.java` | 执行部门 SQL |
| 数据库 | `database/init.sql` 中的 `dept` 表 | 保存部门数据 |

前端 `/api/depts` 经 Vite 代理去掉 `/api` 后到达后端 `/depts`。查询结果经 `Result` 包装成 JSON，再由页面展示。

删除部门是另一个可阅读的例子：Service 会检查部门下是否有关联员工；有关联时拒绝删除。这是现有代码中的规则。

后续个人练习拟从部门名称关键词筛选开始；该需求当前未实现。希望通过一项小改动练习需求拆分、接口参数、SQL 条件和结果检查。

## 数据

`database/init.sql` 包含 8 张表结构，以及虚构部门、教师、班级、学员和工作经历；日志表为空。演示手机号、证件号使用明显无效的占位值，用于说明字段结构。不要将示例当作真实人员资料或真实业务数据。
