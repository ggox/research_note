* 获取当前Maven项目中的classpath
mvn dependency:build-classpath -Dmdep.outputFile=classPath.txt

* 依赖分析
mvn dependency:tree -Dincludes=:spring*:: 

* 下载jar包
mvn dependency:copy-dependencies -DoutputDirectory=src/main/webapp/WEB-INF/lib  

* 生成项目脚手架
mvn archetype:generate

* 下载源码
mvn dependency:sources

* 依赖下载
mvn dependency:get -DgroupId=mysql -DartifactId=mysql-connector-java -Dversion=5.1.36 -DrepoUrl=http://... -Dtransitive=false
