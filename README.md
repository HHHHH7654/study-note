[阿里云云服务器.md](https://github.com/user-attachments/files/31308136/default.md)
将本地网址利用公网Ip＋nginx命令在云服务器上运行网址
在服务器上建立一个远程连接
可以通过win+R->mstsc建立连接
![[Pasted image 20260820175318.png]]

步骤一：确认阿里云服务器公网 IP
![[Pasted image 20260820182500.png]]

步骤二：阿里云安全组开放 80 端口
![[Pasted image 20260820183618.png]]![[Pasted image 20260820184109.png]]![[Pasted image 20260820184613.png]]![[Pasted image 20260820184730.png]]

步骤三：SSH登录服务器
	使用Windows PowerShell、CMD 或 Terminal 
	具体用户名要看你创建 ECS 时的系统镜像和账号配置
	输入：ssh root@公网IP
		1.输入运行成功后进入命令行 root@iZxxxx:~#      说明进入了云服务器
		2.若为Connection  refused
				说明SSH 22端口出现问题->进入步骤二，
				添加规则协议：TCP
				端口：SSH(22)
				访问来源：我的IP
			
			进入服务器执行：sudo systemctl status ssh 或 sudo systemctl status sshd
		3.如果2之后![[Pasted image 20260820194604.png]]
		为Timeout->连接超时通常说明请求根本没有成功到达服务器
			添加规则协议：TCP
			访问来源：85.211.196.97（我的IP）
			访问目的：SSH(22)
			动作：允许
		
		继续执行ssh root@120.26.115.78
		执行后若出现：The authenticity of host ... can't be established.
	     Are you sure you want to continue connecting (yes/no/[fingerprint])?-->yes

步骤四：安装Nginx
	在服务器中的浏览器下载Nginx，解压至C:\nginx
		然后打开服务器上的 PowerShell：cd C:\nginx  ->.\nginx.exe
			如果没有任何报错，没有输出通常是正常的
			如果出现![[Pasted image 20260820202821.png]]
			先Ctrl + C退出进程，接着让当前 PowerShell 回到：PS C:\nginx\nginx-1.30.4>    -->Start-Process .\nginx.exe  或者 start .\nginx.exe     
				这样 Nginx 会作为单独进程启动，不会占住当前 PowerShell。Nginx 官方 Windows 文档的示例同样使用 `start nginx` 启动。
		
		等待1~2秒，检查进程：tasklist | findstr nginx    
			1. 如果出现nginx.exe说明已启动
		     在浏览器中输入;http://127.0.0.1 或者 http://localhost
				出现Welcome to nginx!  则为安装成功
			最后在电脑上的浏览器输入：http://127.0.0.1
				出现Welcome to nginx!，则正常
			
			2.如果无显示，Nginx启动失败
				执行Get-Content .\logs\error.log -Tail 30
	检查 Windows 防火墙。请用“管理员身份”打开 PowerShell
		New-NetFirewallRule -DisplayName "Nginx HTTP 80" -Direction Inbound -Protocol TCP -LocalPort 80 -Action Allow

步骤五：上传项目
	右键项目在终端中运行：tree /F   如果内容太多：tree /F > project-tree.txt
	① 构建热轧钢前端 → ② 上传到服务器 → ③ 替换 Nginx 默认页面 → ④ 公网 IP 打开你的热轧钢系统 → ⑤ 再部署 FastAPI 后端 → ⑥ 配置 `/api` 反向代理
 在PoerShell执行：Get-Content .\frontend\package.json
	 后执行：Get-Content .\backend\requirements.txt
或者在cmd中执行：type .\frontend\package.json
	后执行：type .\backend\requirements.txt
	
定项目启动方式后执行：cd /d C:\Users\liuzhirong\Desktop\计算机设计大赛\steel-scratch-web\frontend
		（在浏览器中下载Node.js）（安装后执行：npm -v）
	检查Node：node -v  -->出现版本号，需要≥v22.13.0（下图报错原因没有 `package.json`）
	在服务器中进入前端目录安装依赖，在cmd中执行：cd /d C:\steel-scratch-web\frontend   -->  执行npm ci  --生成node_modules
	先执行：cd frontend 
	![[Pasted image 20260820204523.png]]
	执行：npm run build  
	![[Pasted image 20260820204459.png]]
	执行：npm run start -- --port 3000 --hostname 127.0.0.1  ：此命令会占据终端
	![[Pasted image 20260820204437.png]]接着连接远程桌面win+R-->mstsc
		显示选项→ 本地资源→ 本地设备和资源→ 详细信息勾选C盘—重新连接
		进入服务器后打开此电脑会有C盘映射
		将项目文件复制在服务器上
			如果项目文件过大：
			1 测试映射.dir \\tsclient\C-->出现目录则成功
				测试项目路径：dir "\\tsclient\C\Users\liuzhirong\Desktop\计算机设计大赛\steel-scratch-web"-->出现类似frontend，backend，docs，README.md  则为成功
			2 掠过不需要的文件内容，执行：.robocopy"\\tsclient\C\Users\liuzhirong\Desktop\计算机设计大赛\steel-scratch-web" "C:\steel-scratch-web" /E /XJ /XD node_modules dist .wrangler .vinext .git .next __pycache__ /XF project-tree.txt steelvision-site.tar.gz /R:2 /W:2  -->Failed : 0结果为这个成功
			![[Pasted image 20260821110131.png]]
			3.执行：dir C:\steel-scratch-web  出现目录
			后执行：dir C:\steel-scratch-web\frontend  出现文件
			执行：dir C:\steel-scratch-web\backend  出现后端结构
		打开新的PowerShell或者cmd，进入Nginx目录：cd C:\nginx\nginx-1.30.4
		先备份原配置：copy .\conf\nginx.conf .\conf\nginx.conf.bak
		记事本打开：notepad .\conf\nginx.conf
			将原：server{}替换成![[Pasted image 20260820211803.png]]
			或者直接替换
			![[Pasted image 20260821111030.png]]
		保存检查Nginx配置：.\nginx.exe -t  ---出现      syntax is ok
											test is successful则成功
			再执行：.\nginx.exe -s reload   无输出
	测试：先在服务器中浏览器输入：http://127.0.0.1    输出为热轧钢网站
			电脑浏览器输入：http://120.26.115.78   输出为热轧钢网站
		如果不是先按Fn+F5强制刷新，无效则执行nginx.exe -t 和 nginx.exe -s reload
				如出现：502 Bad Gateway  说明将：npm run start -- --port 3000 --hostname 127.0.0.1关闭了，重新;cd /d C:\steel-scratch-web\frontend     和   npm run start -- --port 3000 --hostname 127.0.0.1
			如果网页打不开：依次检查：tasklist | findstr nginx   
									netstat -ano | findstr :80
									netstat -ano | findstr :3000
					正常情况：![[Pasted image 20260821111535.png]]![[Pasted image 20260821111755.png]]是正确监听
	步骤六：部署
		基本流程![[Pasted image 20260821111958.png]]
		如果热轧钢网站出现的是HTML，则是.css文件/JS静态资源加载失败
		在服务器中先执行：cd /d C:\steel-scratch-web\frontend
						dir dist\client\assets\*.css-->生成.css文件
				测试：curl.exe -I http://127.0.0.1:3000/assets/index-ABC123.css
			打开：C:\nginx\nginx-1.30.4\conf\nginx.conf
				替换server{}为![[Pasted image 20260821112724.png]]
				此举不在经过Vinext
				/assets/xxx.css
				      ↓
					Nginx
				      ↓
		C:\steel-scratch-web\frontend\dist\client\assets\xxx.css
				说明：`alias` 会把匹配的 URL 前缀替换为指定的本地目录路径；而其他请求仍可以通过 `proxy_pass` 转发给 Vinext
			完整内容：![[Pasted image 20260821113012.png]]
		之后运行：cd /d C:\nginx\nginx-1.30.4
						nginx.exe -t      ok和successfui为成功
				执行：nginx.exe -s reload
			在电脑中打开：http://120.26.115.78
			验证：curl.exe -I http://127.0.0.1/assets/index-Bt_73FYM.css
				成功：HTTP/1.1 200 OK
				失败：404 Not Found
		确认端口：cd /d C:\steel-scratch-web
			检查python：python --version
		执行：type backend\app\main.py
		![[Pasted image 20260821113523.png]]
		进入后端：cd /d C:\steel-scratch-web\backend
			为了不把 Python 包直接装进整个服务器环境，建议给这个项目单独创建虚拟环境：python -m venv .venv   /    py -m venv .venv
			激活：.venv\Scripts\activate
				成功出现![[Pasted image 20260821113418.png]]
			升级pip：python -m pip install --upgrade pip
				安装后端依赖：pip install -r requirements.txt
		输入：uvicorn app.main:app --host 127.0.0.1 --port 8000  成功为以下
		![[Pasted image 20260821113639.png]]新CMD窗口测试后端：curl.exe http://127.0.0.1:8000/health
			返回一个JSON
	后端连接到Nginx，修改Nginx配置
		完整Nginx配置![[Pasted image 20260821113906.png]]执行：cd /d C:\nginx\nginx-1.30.4
					nginx.exe -t
			成功后：nginx.exe -s reload
			执行：type C:\steel-scratch-web\backend\app\routers\inferences.py
			![[Pasted image 20260821114403.png]]测试：curl.exe http://127.0.0.1/api/v1/system/info
			如果为这个 `504 Gateway Time-out` 说明：**Nginx 已经收到 `/api/v1/system/info` 请求了，但它没有及时从 FastAPI 的 `127.0.0.1:8000` 得到响应。**
			![[Pasted image 20260821114636.png]]在服务器中新开cmd窗口：测试curl.exe http://127.0.0.1:8000/api/v1/system/info   -->返回一个JSON
				若失败：netstat -ano | findstr :8000  测试是否有监听LISTENING
						没有依次执行：cd /d C:\steel-scratch-web\backend
						.venv\Scripts\activate
						uvicorn app.main:app --host 127.0.0.1 --port 8000
						![[Pasted image 20260821115311.png]]此为成功
				新开CMD窗口：curl.exe http://127.0.0.1:8000/api/v1/system/info
				直接访问成功接着：curl.exe http://127.0.0.1/api/v1/system/info
				![[Pasted image 20260821115408.png]]在电脑上访问：http://120.26.115.78/api/v1/system/info -->返回JSON
		实现服务器启动后自动运行
			服务器创建目录：C:\steel-scratch-web\scripts
			创建前端启动脚本;notepad C:\steel-scratch-web\scripts\start-frontend.bat
			笔记本写入：![[Pasted image 20260821124035.png]]
			FastAPI后端：notepad C:\steel-scratch-web\scripts\start-backend.bat
			`@echo off`
			`cd /d C:\steel-scratch-web\backend`
			`C:\steel-scratch-web\backend\.venv\Scripts\python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000`
			Nginx：notepad C:\steel-scratch-web\scripts\start-nginx.bat
			![[Pasted image 20260821163905.png]]
			在服务器中搜索“任务计划程序”，选则创建“基本任务”。三个基本设置如下![[Pasted image 20260821163055.png]]![[Pasted image 20260821163145.png]]![[Pasted image 20260821163239.png]]![[Pasted image 20260821163301.png]]![[Pasted image 20260821163209.png]]其中操作中的“程序或者脚本”，“参数”，“起始位置”依情况而定。
			完成后在服务器上测试：netstat -ano | findstr :3000
								netstat -ano | findstr :8000
								netstat -ano | findstr :80
							返回错误：![[Pasted image 20260821164301.png]]
							80    → LISTENING   ✓ Nginx 正在运行
							3000  → 没有 LISTENING ✗ 前端没有运行
							8000  → 没有任何结果   ✗ 后端没有运行
						执行：C:\steel-scratch-web\scripts\start-frontend.bat
						显示成功为：[vinext] Production server running at http://127.0.0.1:3000
						打开新CMD窗口：C:\steel-scratch-web\scripts\start-backend.bat
						显示成功为：Uvicorn running on http://127.0.0.1:8000
						![[Pasted image 20260821164538.png]]
						打开新CMD： 先    netstat -ano | findstr :3000
							        后netstat -ano | findstr :8000
							        成功出现“LISTENING”
			在电脑上测试：curl.exe http://127.0.0.1/api/v1/system/info
						http://120.26.115.78
			在任务计划程序中更改SteelVision Frontend和SteelVision Backend操作：前端任务 → 属性 → “操作” → 编辑：
				程序或脚本：C:\Windows\System32\cmd.exe
				参数：/c C:\steel-scratch-web\scripts\start-frontend.bat
				起始于：C:\steel-scratch-web\frontend
				（后端“frontend”改成“backend”）
			触发器设置成“延迟任务：30秒”
			其余设置与之前相同，设置好后运行
				测试：netstat -ano | findstr :3000
					  netstat -ano | findstr :8000
					  再netstat -ano | findstr :8000
					  成功都出现“ LISTENING”
						  如果8000没有成功
							  后端程序改成：C:\steel-scratch-web\backend\.venv\Scripts\python.exe
							  参数：-m uvicorn app.main:app --host 127.0.0.1 --port 8000
							  起始于：C:\steel-scratch-web\backend
					重新测试监听
					最后测试：curl.exe http://127.0.0.1/api/v1/system/info
						返回JSON
				在服务器中打开：http://127.0.0.1
					电脑打开：http://120.26.115.78
					都出现热轧钢网站
				最后重启服务器，在电脑上：前端网站：http://120.26.115.78
							后端接口：http://120.26.115.78/api/v1/system/info
							如果都显示热轧钢网站则成功
		服务器加密
			在服务器中安全组中新增规则![[Pasted image 20260821185725.png]]
			如果控制台”我的IP“显示有，就选我的IP。删除原来的：![[Pasted image 20260821185854.png]]如果还有下图，则选择删除或者禁用![[Pasted image 20260821185921.png]]第一步，把第 1 条：
					SSH(22)
					85.211.196.97
			点击“编辑”，改成：
					RDP(3389)
					85.211.196.9![[Pasted image 20260821190124.png]]
				确认这条新规则已经存在以后，删除最后一条：
					RDP(3389)
					0.0.0.0/0
					这样远程桌面就不再暴露给整个互联网，只允许你自己的公网 IP。
				删除：
					SSH(22)
					0.0.0.0/0
					![[Pasted image 20260821191812.png]]
					在电脑上PowerShell中运行：Test-NetConnection 120.26.115.78 -Port 3389
					显示：TcpTestSucceeded : True  无问题
			日志 + 服务故障恢复
				进入服务器打开：C:\nginx\nginx-1.30.4\logs
				或者CMD执行：dir C:\nginx\nginx-1.30.4\logs
					出现![[Pasted image 20260821192222.png]]
					正常情况下：  access.log：记录谁访问了你的网站、访问了什么地址、HTTP 状态码等。
								error.log：记录 Nginx 转发失败、配置问题、502/504 等错误。
						![[Pasted image 20260821192413.png]]![[Pasted image 20260821192418.png]]
				创建统一日志目录
					在服务器CMD执行：mkdir C:\steel-scratch-web\logs
									 dir C:\steel-scratch-web\logs
				创建后端日志启动脚本
					打开记事本，新建：C:\steel-scratch-web\scripts\start-backend-logged.bat
						内容：@echo off
								cd /d C:\steel-scratch-web\backend
								echo. >> C:\steel-scratch-web\logs\backend.log
								echo ============================== >> C:\steel-scratch-web\logs\backend.log
								echo SteelVision Backend Start %date% %time% >> C:\steel-scratch-web\logs\backend.log
								echo ============================== >> C:\steel-scratch-web\logs\backend.log
								C:\steel-scratch-web\backend\.venv\Scripts\python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000 >> C:\steel-scratch-web\logs\backend.log 2>&1
				创建前端日志启动脚本：
					服务器CMD：where npm
						结果
					![[Pasted image 20260821193038.png]]
					![[Pasted image 20260821193301.png]]
					创建：C:\steel-scratch-web\scripts\start-frontend-logged.bat
						内容：@echo off
								cd /d C:\steel-scratch-web\frontend
								echo. >> C:\steel-scratch-web\logs\frontend.log
								echo ============================== >> C:\steel-scratch-web\logs\frontend.log
								echo SteelVision Frontend Start %date% %time% >> C:\steel-scratch-web\logs\frontend.log
								echo ============================== >> C:\steel-scratch-web\logs\frontend.log
								call "C:\Program Files\nodejs\npm.cmd" run start -- --port 3000 --hostname 127.0.0.1 >> C:\steel-scratch-web\logs\frontend.log 2>&1
				后端：notepad C:\steel-scratch-web\scripts\start-backend-logged.bat
					内容：@echo off
							cd /d C:\steel-scratch-web\backend
							echo. >> C:\steel-scratch-web\logs\backend.log
							echo ============================== >> C:\steel-scratch-web\logs\backend.log
							echo SteelVision Backend Start %date% %time% >> C:\steel-scratch-web\logs\backend.log
							echo ============================== >> C:\steel-scratch-web\logs\backend.log
							C:\steel-scratch-web\backend\.venv\Scripts\python.exe -m uvicorn app.main:app --host 127.0.0.1 --port 8000 >> C:\steel-scratch-web\logs\backend.log 2>&1
				前端日志：notepad C:\steel-scratch-web\scripts\start-frontend-logged.bat
					内容：@echo off
							cd /d C:\steel-scratch-web\frontend
							echo. >> C:\steel-scratch-web\logs\frontend.log
							echo ============================== >> C:\steel-scratch-web\logs\frontend.log
							echo SteelVision Frontend Start %date% %time% >> C:\steel-scratch-web\logs\frontend.log
							echo ============================== >> C:\steel-scratch-web\logs\frontend.log
							call "C:\Program Files\nodejs\npm.cmd" run start -- --port 3000 --hostname 127.0.0.1 >> C:\steel-scratch-web\logs\frontend.log 2>&1
				如果显示”已存在“，选择替换里面的内容
					确认存在：dir C:\steel-scratch-web\scripts
					![[Pasted image 20260821193744.png]]
					![[Pasted image 20260821193755.png]]
			打开服务器的”任务计划程序“
				结束运行：SteelVision Frontend 和 SteelVision Backend
				在CMD：netstat -ano | findstr :3000 和 netstat -ano | findstr :8000
					此时都没有”LISTENING“
				修改`SteelVision Frontend`
					程序或脚本：C:\Windows\System32\cmd.exe
					添加参数：/c C:\steel-scratch-web\scripts\start-frontend-logged.bat
					起始于：C:\steel-scratch-web\frontend
				修改`SteelVision Backend`
					程序或脚本：C:\Windows\System32\cmd.exe
					添加参数：/c C:\steel-scratch-web\scripts\start-backend-logged.bat
					起始于：C:\steel-scratch-web\backend
				修改后运行，检查监听，结果有”LISTENING“
					检查日志文件是否生成：dir C:\steel-scratch-web\logs-->出现.log
					检查前端日志：type C:\steel-scratch-web\logs\frontend.log
					![[Pasted image 20260821195450.png]]
					检查后端日志：type C:\steel-scratch-web\logs\backend.log
					![[Pasted image 20260821195518.png]]
				最后电脑打开：http://120.26.115.78
							http://120.26.115.78/api/v1/system/info
					正常则完成
		服务异常后的自动恢复：配置 `SteelVision Frontend` 和 `SteelVision Backend` 两个计划任务。打开：任务计划程序→ 任务计划程序库
		![[Pasted image 20260821200526.png]]
		触发器则是：”延迟任务30秒“
			Nginx：“延迟任务15秒”
		故障模拟测试后端：netstat -ano | findstr :8000![[Pasted image 20260821200724.png]]
		在”任务计划程序“中结束运行后端
			再执行：netstat -ano | findstr :8000
			此时无”LISTENING“，运行后再执行一次则有”LISTENING“
		日志容量控制：
			 创建 Nginx 日志归档目录：mkdir C:\nginx\nginx-1.30.4\logs\archive
			 创建 PowerShell 脚本：notepad C:\steel-scratch-web\scripts\rotate-nginx-logs.ps1
			 内容：![[Pasted image 20260821201357.png]]
			 作用：
			![[Pasted image 20260821202240.png]]
			测试管理员CMD：powershell -ExecutionPolicy Bypass -File C:\steel-scratch-web\scripts\rotate-nginx-logs.ps1
				再执行：dir C:\nginx\nginx-1.30.4\logs
			![[Pasted image 20260821202329.png]]
			执行：dir C:\nginx\nginx-1.30.4\logs\archive
				![[Pasted image 20260821202424.png]]
				在电脑测试：  http://120.26.115.78
							http://120.26.115.78/api/v1/system/info
			在计划任务程序中创建任务：
				名称：SteelVision Nginx Log Rotate
				![[Pasted image 20260821202613.png]]
				触发器：
				![[Pasted image 20260821202631.png]]
				操作：
					程序或脚本：powershell.exe
					参数：-ExecutionPolicy Bypass -File "C:\steel-scratch-web\scripts\rotate-nginx-logs.ps1"
					起始于：C:\steel-scratch-web\scripts
				设置：![[Pasted image 20260821202738.png]]
				完成后运行，检查：dir C:\nginx\nginx-1.30.4\logs\archive-->.log
			创建：notepad C:\steel-scratch-web\scripts\rotate-app-logs.ps1
				内容：![[Pasted image 20260821204608.png]]
				在管理员PoerShell中执行：Get-ScheduledTask -TaskName "SteelVision Frontend"
				再：Get-ScheduledTask -TaskName "SteelVision Backend"
				继续创建日志轮转脚本：notepad C:\steel-scratch-web\scripts\rotate-app-logs.ps1
				内容：
				![[Pasted image 20260821204044.png]]
				保存执行：powershell -ExecutionPolicy Bypass -File "C:\steel-scratch-web\scripts\rotate-app-logs.ps1"
				检查监听：netstat -ano | findstr :3000 和 netstat -ano | findstr :8000
				![[Pasted image 20260821204633.png]]
				检查日志：dir C:\steel-scratch-web\logs -->.log
				检查归档：dir C:\steel-scratch-web\logs\archive-->xxx.xxx.log
			电脑测试：http://120.26.115.78
					  http://120.26.115.78/api/v1/system/info
		在计划程序任务中创建：SteelVision App Log Rotate![[Pasted image 20260821204946.png]]![[Pasted image 20260821205000.png]]
		操作
			程序或脚本：powershell.exe
			参数：-ExecutionPolicy Bypass -File "C:\steel-scratch-web\scripts\rotate-app-logs.ps1"
			起始于：C:\steel-scratch-web\scripts
			![[Pasted image 20260821205119.png]]
			保存后运行
			确认是否监听：netstat -ano | findstr :3000 和 netstat -ano | findstr :8000
			执行：dir C:\steel-scratch-web\logs\archive-->xxx.xxx..log
			电脑访问：http://120.26.115.78
					  http://120.26.115.78/api/v1/system/info
	设置自动备份：
		服务器管理员CMD：mkdir C:\steelvision-backup
						 dir C:\steelvision-backup
		创建自动备份脚本：notepad C:\steel-scratch-web\scripts\backup-steelvision.ps1
			内容：`$stamp = Get-Date -Format "yyyyMMdd-HHmmss"`
				`$backupRoot = "C:\steelvision-backup"`
				`$tempRoot = "$backupRoot\SteelVision-$stamp"`
				`$project = "C:\steel-scratch-web"`
				`$nginx = "C:\nginx\nginx-1.30.4"`
				`New-Item -ItemType Directory -Force -Path $tempRoot | Out-Null`
				`Write-Host "Creating SteelVision backup..."`
				`# 1. Nginx 配置`
				`New-Item -ItemType Directory -Force -Path "$tempRoot\nginx" | Out-Null`
				`Copy-Item `
				    `"$nginx\conf" `
				    `"$tempRoot\nginx\conf" `
				    `-Recurse `
				    `-Force`
				`# 2. SteelVision 脚本`
				`Copy-Item `
				    `"$project\scripts" `
				    `"$tempRoot\scripts" `
				    `-Recurse `
				    `-Force`
				`# 3. 后端源代码`
				`#    不备份 .venv 和缓存`
				`New-Item -ItemType Directory -Force -Path "$tempRoot\backend" | Out-Null`
				`robocopy `
				    `"$project\backend" `
				    `"$tempRoot\backend" `
				    `/E `
				    `/XD .venv __pycache__ `
				    `/XF *.pyc `
				    `/R:1 `
				    `/W:1 | Out-Null`
				`# 4. 前端源代码`
				`#    排除可重新生成的目录`
				`New-Item -ItemType Directory -Force -Path "$tempRoot\frontend" | Out-Null`
				`robocopy `
				    `"$project\frontend" `
				    `"$tempRoot\frontend" `
				    `/E `
				    `/XD node_modules dist .next .wrangler .git `
				    `/R:1 `
				    `/W:1 | Out-Null`
				`# 5. 项目文档`
				`if (Test-Path "$project\docs") {`
				    `Copy-Item ```
				        `"$project\docs" ```
				        `"$tempRoot\docs" ```
				        `-Recurse ```
				        `-Force`
				`}`
				`# 6. 根目录重要文件`
				`$rootFiles = @(`
				    `"README.md",`
				    `"project-tree.txt"`
				`)`
				`foreach ($file in $rootFiles) {`
				    `if (Test-Path "$project\$file") {`
				        `Copy-Item ```
				            `"$project\$file" ```
				            `"$tempRoot\$file" ```
				            `-Force`
				    `}`
				`}`
				`# 7. 导出 Windows 计划任务`
				`$taskFolder = "$tempRoot\tasks"`
				`New-Item -ItemType Directory -Force -Path $taskFolder | Out-Null`
				`$tasks = @(`
				    `"SteelVision Frontend",`
				    `"SteelVision Backend",`
				    `"SteelVision Nginx",`
				    `"SteelVision Nginx Log Rotate",`
				    `"SteelVision App Log Rotate"`
				`)`
				`foreach ($task in $tasks) {`
				    `try {`
				        `Export-ScheduledTask -TaskName $task |`
				            `Out-File ```
				                `"$taskFolder\$($task.Replace(' ','-')).xml" ```
				                `-Encoding UTF8`
				        `Write-Host "Exported task: $task"`
				    `}`
				    `catch {`
				        `Write-Host "Task not found: $task"`
				    `}`
				`}`
				`# 8. 压缩备份`
				`$zipFile = "$backupRoot\SteelVision-backup-$stamp.zip"`
				`Compress-Archive `
				    `-Path "$tempRoot\*" `
				    `-DestinationPath $zipFile `
				    `-Force`
				`# 删除临时目录`
				`Remove-Item `
				    `$tempRoot `
				    `-Recurse `
				    `-Force`
				`# 9. 删除30天以前的备份`
				`Get-ChildItem `
				    `$backupRoot `
				    `-Filter "SteelVision-backup-*.zip" |`
				    `Where-Object {`
				        `$_.LastWriteTime -lt (Get-Date).AddDays(-30)`
				    `} |`
				    `Remove-Item -Force`
				`Write-Host ""`
				`Write-Host "================================="`
				`Write-Host "SteelVision backup completed."`
				`Write-Host "Backup file:"`
				`Write-Host $zipFile`
				`Write-Host "================================="`
		测试备份：powershell -ExecutionPolicy Bypass -File "C:\steel-scratch-web\scripts\backup-steelvision.ps1"
		![[Pasted image 20260821210525.png]]
			 执行：dir C:\steelvision-backup-->出现一个压缩包
		任务计划程序创建任务：SteelVision Weekly Backup
			![[Pasted image 20260821210804.png]]
			操作：程序或脚本：powershell.exe
				参数：-ExecutionPolicy Bypass -File "C:\steel-scratch-web\scripts\backup-steelvision.ps1"
				起始于：C:\steel-scratch-web\scripts
				![[Pasted image 20260821210906.png]]
				运行
				检查：dir C:\steelvision-backup-->出现压缩包
		域名：![[Pasted image 20260821211044.png]]
		注册完域名后找到“云解析DNS”添加
			![[Pasted image 20260821211215.png]]![[Pasted image 20260821211224.png]]
			电脑执行：nslookup smartsteel.site
				再执行：nslookup www.smartsteel.site  两次都是地址
			在服务器 CMD 备份当前 Nginx 配置
			打开配置文件：notepad C:\nginx\nginx-1.30.4\conf\nginx.conf
				将“server_name  _;”改成 “server_name  smartsteel.site www.smartsteel.site;”
				保存后检查：cd /d C:\nginx\nginx-1.30.4 和 nginx.exe -t
				成功后执行：nginx.exe -s reload
					服务器内：curl.exe -I -H "Host: smartsteel.site" http://127.0.0.1
			如果出现是否需要覆盖选择N，建立新的备份文件：copy C:\nginx\nginx-1.30.4\conf\nginx.conf C:\nginx\nginx-1.30.4\conf\nginx.conf.bak-20260820
					在电脑上测试：![[Pasted image 20260821211754.png]]
			申请个人测试证书
			ICP备案
