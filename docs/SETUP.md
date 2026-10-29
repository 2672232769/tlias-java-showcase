# 本地运行与构建

## 本地运行

准备 JDK 17 或 21、Maven 3.9、MySQL 8，以及 Node.js 20 和 npm。下面以 Windows PowerShell 为例。仓库公开，无需 GitHub 登录即可浏览和获取代码。

### 1. 获取代码

```powershell
git clone https://github.com/2672232769/tlias-java-showcase.git
cd tlias-java-showcase
```

### 2. 初始化独立练习数据库

确保 MySQL 已启动。在仓库根目录执行：

```powershell
mysql --default-character-set=utf8mb4 -u root -p
```

进入 MySQL 后，创建一个**全新的空数据库**再导入：

```sql
CREATE DATABASE tlias_learning CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci;
USE tlias_learning;
SOURCE database/init.sql;
EXIT;
```

如 `tlias_learning` 已存在，请改用另一个空练习库，并相应设置下一步的 `DB_URL`。初始化脚本只用于首次导入，不要重复导入到已有业务库。脚本不含删除数据库或删除表的语句。

脚本包含 8 张课程表结构，以及少量重新编写的虚构部门、教师、班级、学员和工作经历数据；日志表初始为空。这里不是个人本机数据库备份，也没有导入课程 SQL 中的历史日志和账号记录。

### 3. 启动后端

在仓库根目录的 PowerShell 中设置环境变量：

```powershell
$env:DB_URL = 'jdbc:mysql://127.0.0.1:3306/tlias_learning'
$env:DB_USERNAME = 'root'
$dbPassword = Read-Host '请输入本机 MySQL 密码' -AsSecureString
$env:DB_PASSWORD = [System.Net.NetworkCredential]::new('', $dbPassword).Password

# 生成本次本地练习使用的 JWT 签名密钥，不要把它提交到 Git。
$jwtBytes = New-Object byte[] 32
$jwtRng = [System.Security.Cryptography.RandomNumberGenerator]::Create()
$jwtRng.GetBytes($jwtBytes)
$jwtRng.Dispose()
$env:JWT_SECRET_BASE64 = [Convert]::ToBase64String($jwtBytes)

cd backend
mvn spring-boot:run
```

后端默认监听 `http://127.0.0.1:8080`。保持该终端运行。环境变量只作用于当前终端及其启动的程序；换终端启动时需重新设置。重新生成 JWT 密钥后，旧登录令牌会失效，重新登录即可。

若用 IDEA 启动，也要在运行配置中设置上述环境变量。项目不会自动加载 `.env` 文件。

### 4. 启动前端

另开一个终端，在仓库根目录执行：

```powershell
cd frontend
npm ci
npm run dev -- --host 127.0.0.1 --port 5173
```

打开 **http://127.0.0.1:5173/login**。

仅针对上面提供的虚构演示数据，登录账号为 `admin`，密码为 `123456`。这个账号是本地课程演示账号，不是 MySQL 账号。

前端通过 Vite 将 `/api` 转发到 `http://localhost:8080`，转发时去掉 `/api`。如果修改后端端口，还需要同步修改 `frontend/vite.config.js` 中的代理地址。

### 5. 图片上传（可选）

登录、查询部门和员工等接口不要求 OSS 凭证；使用图片上传前，在**启动后端的同一个终端**配置自己的：

| 环境变量 | 用途 |
| --- | --- |
| `OSS_ACCESS_KEY_ID` | 阿里云访问凭证 ID |
| `OSS_ACCESS_KEY_SECRET` | 阿里云访问凭证密钥 |
| `OSS_ENDPOINT` | 对应地域的 OSS Endpoint |
| `OSS_BUCKET_NAME` | 自己的存储桶名称 |
| `OSS_REGION` | 对应地域，例如 `cn-beijing` |

修改后重启后端。还需按自己的存储桶配置图片访问权限；未配置时上传会失败。仓库不提供可用的 OSS 账号或存储桶。

## 构建与检查

后端打包（跳过课程测试执行）：

```powershell
cd backend
mvn -DskipTests package
```

生成 `backend/target/tlias-web-management-0.0.1-SNAPSHOT.jar`，可在已设置上述环境变量的终端中用 `java -jar` 启动。

前端构建：

```powershell
cd frontend
npm ci
npm run build
```

输出位于 `frontend/dist/`。这是静态构建产物；单独打开 HTML 文件或只启动静态预览不等于完整部署，还需要后端和 `/api` 反向代理。日常练习建议先使用上面的 Vite 开发模式。

仓库保留了课程中的 JWT、日志、ThreadLocal 等演示测试。其中存在固定的历史 JWT，直接运行全部测试可能因令牌过期或缺少数据库配置而失败；它们不是完整的业务回归测试套件。

