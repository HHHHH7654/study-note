一.环境安装
	1.前置环境安装miniconda
		法一：清华镜像网站
		https://mirrors.tuna.tsinghua.edu.cn/anaconda/minicinda
			安装在无中文目录
			添加在PATH
		法二：解决网络问题，官网下载
			https://www.anaconda.com/
			安装在无中文目录
			添加在PATH
				Pytorch
			安装历史版本，电脑版本可以适配的
	安装Pytorch
		安装历史版本，电脑版本可以适配的
	安装CUDA
	2.conda环境创建
		在anconda promat命令
			conda create -n yolov8 python=3.xx 指定版本，版本不宜过高）
	3.pypi配置国内源
		清华源：[https://mirrors.](https://mirrors.tuna.tsinghua.edu.cn/help/pypi/
	4.yolov8安装
		pip源码安装（推荐）
			github：https://github.com/ultralytics
			&& ultralytics-main.zip
			&&cd ultralytics-main
			pip install -e .(-e参数必须要有，否则后续修改代码无效)
		pip直接安装（官方推荐）
			pip install ultralytics
二.使用miniconda
	在windows搜索中搜索“anaconda promat"
		先指定安装python版本环境
		输入：conda activate yolov8   --激活
		code . ---进入vscode并进入yolov8环境
		cd Desktop/ultralytics-main --进入ultralytic
		yolo predict model=yolov8n.pt source=ultralytics/assets/bus.jpg
		（下图中为结果）
		在predict函数中添加“print("代码执行到这里了”)"表示对源码的修改
		![[Pasted image 20260826200221.png]]
三.数据集准备
	1）.images:存放图片
		train：训练集图片
		val：验证集图片
	2）.labels：存放标签
		train：训练集标签文件，要与训练集图片一一对应
		val：验证集标签文件，要与验证集图片一一对应
	新建文件夹“bvn"，将以上两个放入，在ultralytics-main中创建新文件夹“datasets”，将bvn放入此文件夹     注意：yolov8数据集只能放入“datasets”
	3）数据描述文件
		与yolov5不同，path不在是从项目根目录写起，而是从datasets文件夹写起
		![[Pasted image 20260827202103.png]]
四.模型训练
	命令行;yolo task=detect mode=./yolov8n.pt data="data.yaml" workers=1 epochs=50 batch=16
	![[Pasted image 20260827210621.png]]
![[屏幕截图 2026-08-27 215436.png]]
...........................................................................................
![[Pasted image 20260827215622.png]]
![[Pasted image 20260827221920.png]]
![[Pasted image 20260827221936.png]]
	代码启动：需要在python 3.xxx(yolov8)环境中
	![[Pasted image 20260827220127.png]]
		若报错，将“worker=1”改成“worker=0”
	配置文件快捷使用
		复制配置文件：yolo copy-cfg         yolo cfg=default-copy.yaml
			执行完目录多一个“default_copy.yaml"目录
				此时可以修改需要的参数model，data，epoch，batch，workers
					在终端输入yolo cfg=default_copy.yaml开始训练
	修改对应参数：
		model:指定要训练的模型或预训练权重。比如 `yolov8n.pt`，表示使用 YOLOv8 Nano 预训练模型。
		data:指定数据集配置文件，也就是你的 `data.yaml` 路径。比如 `datasets/bvn/data.yaml`。
		epoch:训练轮数。更常见的参数名是 `epochs`。例如 `epochs=50` 表示完整训练数据集 50 轮。
		-batch:每次送进模型训练的图片数量。比如 `batch=16` 表示每个 batch 处理 16 张图。显存不够时可以改成 8 或 4。
		-workers:数据加载进程数。比如 `workers=1`，表示用 1 个 worker 读取训练图片和标签。Windows 下为了减少多进程报错，初次训练用 `workers=0` 或 `1` 都比较常见。
	终端中输入yolo detect predictmodel=runs/detect/train/weigths/best.pt  source=./+视频名字  show=Ture   ---检查预测效果和可视化
常见问题：
	以代码的形式运行时：
		workers 要设置成 0（windows上）
	页面文件太小，无法完成操作：
		调整训练参数中的workers，设置为 1/0
		调整虚拟内存，将环境安装位置所在的盘，设置一个较大的参数
	数据集描述文件：
		数据地址从datasets目录里开始写起，且就放在根目录下，会避免许多坑
	调整数据集目录后再次训练：
		删除~/AppData/Romaing/Ultralytics文件夹下的settings.yaml
	
	