# calpuff编译手册（以及calwrf、calmet和calpost）

## 系统版本
	CentOS Linux release 7.6.1810 (Core)

## Requirement
	#安装需要
	make
	gfrotran 7.3
	NetCDF 版本大于 4.4.0 (CALWRF需要 only）

	#运行需要
	awk
	gdal
	
## 更新yum源（出现无法安装gcc的问题时）
	#在本次编译环境中本地yum源会出现依赖库版本冲突无法安装gcc的问题
	#使用downgrade也无法通过
	wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
	sed -i  's/$releasever/7/g' /etc/yum.repos.d/CentOS-Base.repo
	yum repolist

## 安装gcc和gcc-c++
	yum install gcc
	yum install gcc-c++
	#此时会安装老版本gfortran 4，该版本无法编译calpuff
	gfortran --version 
	#版本为
	GNU Fortran (GCC) 4.8.5 20150623 (Red Hat 4.8.5-44)

## 安装centos-release-scl-rh和devtoolset-7-toolchain
	yum install centos-release-scl-rh
	yum install devtoolset-7-toolchain
	#安装该库是为了得到gfortran 7

## 安装gdal和awk
	yum install -y gdal gdal-devel gawk

## 切换gfortran版本
	scl enable devtoolset-7 bash
	
	gfortran --version 
	#版本为
	GNU Fortran (GCC) 7.3.1 20180303 (Red Hat 7.3.1-5)
	
## 配置环境变量
	export PATH=/public/software/mathlib/netcdf/4.4.1/intel/bin:$PATH

	export LD_LIBRARY_PATH=/public/software/compiler/intel/composer_xe_2015.2.164/compiler/lib/intel64:$LD_LIBRARY_PATH

	which ncdump
	#应为
	/public/software/mathlib/netcdf/4.4.1/intel/bin/ncdump

## 下载zip
	wget http://www.src.com/calpuff/download/Mod7_Files/CALWRF_v2.0.3_L190426.zip
	
	wget http://www.src.com/calpuff/download/Mod7_Files/CALMET_v6.5.0_L150223.zip
	
	wget http://www.src.com/calpuff/download/Mod7_Files/CALPUFF_V7.2.1_L150618.zip
	
	wget http://www.src.com/calpuff/download/Mod7_Files/CALPOST_v7.1.0_L141010.zip
	
	#解压
	unzip *zip

## 编译CALWRF
	#todo

## 编译CALMET
	#todo

## 编译CALPUFF
	cd CALPUFF_v7.2.1_L150618
	
	#需要将所有大写的文件名改为小写，分别执行：
	echo "for f in `find .`; do mv -v "$f" "`echo $f | tr '[A-Z]' '[a-z]'`"; done" > rename.csh
	chmod +x rename.csh
	./rename.csh

	#修改后运行
	gfortran modules.for calpuff.for -o calpuff.exe

	#会出现一堆warning，往上拉可以看到报错项
	Error: Nonnegative width required in format string at (1)

	#进入calpuff.for修改代码
	vi calpuff.for
	#逐项对报错read()函数所在行进行修改
	#例1，将：
	read(line_ver((i1+4):32),'(f)') rver 
	#修改为 
	read(line_ver((i1+4):32),*) rver ,
	
	#例2，将：
	read(awork2(1:n2),'(i)') irmap(nsamp) 
	#修改为 
	read(awork2(1:n2),*) irmap(nsamp)
	
	#再执行
	gfortran modules.for calpuff.for -o calpuff.exe
	#成功时将生成
	calpuff.exe

## 编译CALPOST
	#运行
	gfortran calpost.for -o calpost.exe

	#会出现一堆warning，往上拉可以看到报错项
	Error: Nonnegative width required in format string at (1)
	
	#进入calpost.for修改代码
	参考编译CALPUFF过程

	#再执行
	gfortran calpost.for -o calpost.exe
	#成功时将生成
	calpost.exe