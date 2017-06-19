###引言


&nbsp;&nbsp;&nbsp;&nbsp;github Pages可以被认为是用户编写的、托管在github上的静态网页,而Jekyll是一个静态站点生成器，它会根据网页源码生成静态文件，由github page 和 jekyll 配合使用，能搭建一个免费，不限流的在线个人博客

###搭建环境

+ ruby & ruby gem
+ github 账号
+ liunx or mac (最好)
+ 集成markdown环境的IDE

###搭建步骤

* ####Github Page 创建

	1. 登录github账号，在首页点击 “New repository” 新建库
	2. 在Repository name下输入username.github.io---(username为你的github账号的名称)
	3. 然后按“create repository”，至于后续怎么初始化，怎么提交分支，查阅git的基本操作好了

	**注意：github page 有两种形式的域名User or organization site和project site，user site 相对来说搭建比较简单，但是一个账号只能创建一个**

* ####Jekyll 搭建

	*推荐使用rvm，jekyll的安装要求必须使用最新版本的gem，如果用例如CentOS中yum命令安装，可能导致gem是老版本的*

	1. 安装rvm	

			curl -L get.rvm.io | bash -s stable
			添加source /etc/profile.d/rvm.

		**如果提示安装出错，可能是有些依赖包缺失，用yum -y install package**

	2. 安装ruby & ruby gem

			rvm install 2.2.1

		**rvm list known查看所有版本**

	3. jekyll的安装

			gem install jekyll

	4. 项目创建

		* 从刚刚创建的github项目中创建

				git clone https://github.com/username/username.github.io
				cd username.github.io
				jekyll new . --force

		* 新建一个项目

				jekyll new myblog
				cd myblog

	5. 目录结构介绍
		新创建完了jekyll项目以后，目录结构如下

		重要的目录和文件说明

		* _config.yml --- jekyll环境的配置文件
		* index.html  --- 项目的首页
		* _includes   --- 包含页头和页脚的可重复利用在模板以及页面中的部分html
		* _layouts    --- 项目的模板
		* _posts      --- 文章存放的地方，以Y-m-d-title.ext的形式存放

	6. 一般开发步骤

		1. 在_includes目录下创建好一般网页公共的header，footer的html
		2. 使用**{% include file.ext %}**的形式创建一个网站的模板
		3. 在index.html和_posts文件夹下使用YAML头包含进模板

	7. 编译 & 运行 

		* 一般编译 & 运行方法

				jekyll build
				jekyll serve

		* 动态编译

				jekyll serve --watch
