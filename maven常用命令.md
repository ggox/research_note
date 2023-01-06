# 获取当前Maven项目中的classpath
mvn dependency:build-classpath -Dmdep.outputFile=classPath.txt

# mvn jar包冲突监测
mvn dependency:tree
