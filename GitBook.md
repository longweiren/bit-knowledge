### 使用GitBook

##### 安装

	$ npm install -g gitbook-cli

##### 使用

	$ gitbook help
	build [book] [output]      build a book
    	--format      Format to build to (Default is website; Values are website, json, ebook)
    	--log      Minimum log level to display (Default is info; Values are debug, info, warn, error, disabled)

	pdf [book] [output]      build a book to pdf
    	--log      Minimum log level to display (Default is info; Values are debug, info, warn, error, disabled)

	epub [book] [output]      build a book to epub
    	--log      Minimum log level to display (Default is info; Values are debug, info, warn, error, disabled)

	mobi [book] [output]      build a book to mobi
    	--log      Minimum log level to display (Default is info; Values are debug, info, warn, error, disabled)

	serve [book]      Build then serve a gitbook from a directory
    	--port      Port for server to listen on (Default is 4000)
    	--lrport      Port for livereload server to listen on (Default is 35729)
    	--watch      Enable/disable file watcher (Default is true)
    	--format      Format to build to (Default is website; Values are website, json, ebook)
    	--log      Minimum log level to display (Default is info; Values are debug, info, warn, error, disabled)

	install [book]      install plugins dependencies

	init [directory]      create files and folders based on contents of SUMMARY.md


	gitbook init:初始化书籍目录 
	gitbook serve: 本地启动一个服务器，自动编译并更新内容，在浏览器中预览

##### 发布到 GitHub Pages