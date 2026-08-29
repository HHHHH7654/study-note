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
		删除~/AppData/Romaing/Ultralytics文件夹下的settings.yaml（可以删除settings.yaml,再执行copy）
五.模型预测
	1）模型预测的基本使用
		yolo task=detect model=predict model=/.yolov8n.pt source="./ultralytics/assets/+图片名.格式![[Pasted image 20260828104334.png]]
		 下图为终端直接输入yolo predict model=./yolov8n.pt source=./ultralytics/assets/bus.jpg![[Pasted image 20260828104315.png]]
		 下图为直接代码运行![[Pasted image 20260828104302.png]]
		 对视频进行检测![[Pasted image 20260828104246.png]]![[Pasted image 20260828105501.png]]
		 开启摄像头进行检测![[Pasted image 20260828105400.png]]![[Pasted image 20260828105439.png]]
		保存结果![[Pasted image 20260828105604.png]]
		关键参数
			conf：**conf = confidence threshold，置信度阈值**
				默认值：0.25（推理 / 预测）；训练时默认不使用这个参数
				作用：**过滤掉置信度低于该数值的检测框**。只有模型认为可信度 ≥ conf 的框才会被保留输出。![[Pasted image 20260828105722.png]]
				conf=0.4![[Pasted image 20260828110250.png]]
			iou:：**NMS (非极大值抑制) 的交并比阈值，默认 0.7（只作用于推理预测阶段，不参与训练）
				 作用：同一个物体经常会预测出好几个重叠的框。NMS 用来删掉重复框。iou 就是判断两个框算不算 “重复重叠” 的标准。
				 IoU (交并比)：两个检测框重叠面积 / 总共占有的面积，取值 0~1。
	2）检测结果拓展
		Boxes信息：保存一张图所有检测框全部信息（坐标、置信度、类别），是 torch 张量对象
			结果可视化matplotlib
				要将BGR转化为RGB
					`plt.imshow(reults[0].plt()[:, :, ::-1])`
					`plt.imshow(cv2.cvtColor(frame, cv2.COLOR_BGR2RGB))`
		![[Pasted image 20260828111227.png]]![[Pasted image 20260828111432.png]]
			`转 numpy 数组（脱离 torch 张量）`
			`xyxy = boxes.xyxy.cpu().numpy()`
			`conf = boxes.conf.cpu().numpy()`
			`cls = boxes.cls.cpu().numpy()`
			`results[0].boxes.xywh.cpu().numpy()`
	注意事项
		Jupyter中使用，要可视化预测模型结果，一定要设置%matplotlib inline ，否则无法使用，因为在plot部分，YOLO强制将后端设置为了Agg
		Jupyter中使用，要重新加载模型，否则预测过程中的参数，将不会更新，除非手动再次给出 

六.数据集构建
	1）数据准备
		图片类型数据
			无需额外处理，直接进行标注
		视频类型数据
			进行抽帧处理，导出为图片
			`video = cv2.VideoCapture("./BVN.mp4")`
			`num = 0`
			`save_step = 30`
			`while True:`
			    `ret, frame = video.read()`
			    `if not ret :`
			        `break`
			    `num += 1`
			    `if num % save_step == 0:`
			        `cv2.imwrite("./demo_images/" + str(num) +".jpg",frame)`
			        ![[Pasted image 20260828230052.png]]
	2)labelimg数据集标注
		环境安装：pip install labelimg
		启动命令：labelimg
			![[Pasted image 20260828231951.png]]
			打开：Open Dir --选择文件夹---再次选择文件夹放入标注结果（可以新建）-----x选择yolo ----- 右键create进行画框标注
		关键设置：autosave
			YOLO format
		数据集构建
			make sense（可以选择训练好的YOLO进行辅助标注，目前只支持yolov5,需要转换）
				https://www.makesense.ai/
				![[Pasted image 20260828235617.png]]![[Pasted image 20260828235628.png]]![[Pasted image 20260828235728.png]]
				标注完成后点击左上角的行动，选择第一个![[Pasted image 20260828235851.png]]![[Pasted image 20260829000055.png]]
			辅助标注
				转换下载：pip install tensorflowjs2.8.5
						 YOLOv5 模型导出tfjs
							 python export,py --weight runs/train/exp2/weigths/best.pt --include tfjs
								 在导出的文件内新建一个标注文件“classes.txt”，里面写标注
				make sense 上传模型
		robofolw公开数据集
			https://public.robflow.com/object-detection
			https://universe.roboflow.com/
拓展
	深度学习经典检测阶段
		Two-stage （两阶段）: Faster-rcnn Mask-Rcnn系列
			优势：速度通常比较慢（5FPS）
		one-stage （单阶段）：YOLO系列
			优势：速度快，适合实时监测任务（越快效果越差，越慢效果越好）
			缺点：效果一般
		指标分析
			map：综合衡量检测效果
			IOU：值越高说明效果越好
				预测值(Prediction)和真实值(Ground truth)
				IoU=Area of Overlap（交集）/Area of Union（并集）
			Precision=TP/(TP+FP)  ------精度
			Reacll=TP/(TP+FN) ------召回率
				举例![[Pasted image 20260829093959.png]]
				TP：正类判定为正类            FP：负类判定为正类
				FN：正类判定为负类           TN：负类判定为负类
				![[Pasted image 20260829095800.png]]![[Pasted image 20260829095748.png]]
		网络架构
			（全连接层需要固定图片大小）![[Pasted image 20260829102558.png]]
			对于预测最终结果$（S*S)*(B*5+C)$  5:x,y,w,h,c
		损失函数：![[Pasted image 20260829141654.png]]
		NMS：非极大值抑制---取IoU最大的 
		K-means聚类中的距离：d(box,centroids)=1-IOU(box,centroids)
	YOLOV2版本舍弃了Dropout，卷积后全部加入了Batch Normalization