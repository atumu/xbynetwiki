title: npm_pakcage.json依赖版本符号的含义 

#  npm pakcage.json依赖版本符号的含义 
http://stackoverflow.com/questions/22343224/difference-between-tilde-and-caret-in-package-json
http://blog.163.com/sdhhqb@126/blog/static/6370553720153147217298/
http://cnodejs.org/topic/532af18e2e21a86d5b00c7fb

  * ~version "Approximately equivalent to version" 
  * ^version "Compatible with version" 
  * version Must match version exactly
  * >version Must be greater than version
  * >=version etc
  * <version
  * <=version
  * 1.2.x 1.2.0, 1.2.1, etc., but not 1.3.0
  * http://sometarballurl (this may be the URL of a tarball which will be downloaded and installed locally
  * * Matches any version
  * latest Obtains latest release

**npm 安装参数中的 –save-dev 是什么意思**
npm install takes 3 exclusive, optional flags which save or update the package version in your main** package.json**:
  * -S, –save: Package will appear in your dependencies.
  * -D, –save-dev: Package will appear in your devDependencies.
  * -O, –save-optional: Package will appear in your optionalDependencies.
-save和-save-dev
**可以省掉你手动修改package.json文件的步骤。**
install module-name -save
自动把模块和版本号添加到dependencies部分
install module-name -save-dve
自动把模块和版本号添加到devdependencies部分
至于配置文件区分这俩部分， 是用于区别开发依赖模块和产品依赖模块
