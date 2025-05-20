1.环境准备，下载java、maven
     安装 JDK 8 和 Maven：sudo yum install -y java-1.8.0-openjdk-devel maven

sudo yum install -y java-1.8.0-openjdk-devel maven
验证
mvn -version | grep "Apache Maven" && java -version | grep "1.8.0"
2.创建项目目录结构和Java应用示例
操作目的：为 Java 应用创建符合 Maven 标准的目录结构，便于编译和管理，创建一个简单的控制台应用，输出 "Hello, Container World!"。
mkdir -p：递归创建目录，即使父目录不存在。
~/my-java-app：项目根目录。
src/main/java/hello：Java 源代码路径，包名为 hello。
cat <<EOF > ...：通过 Here Document 写入文件内容。
package hello;：声明类属于 hello 包。


mkdir -p ~/my-java-app/src/main/java/hello
cat <<EOF > ~/my-java-app/src/main/java/hello/HelloWorld.java
package hello;
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello, Container World!");
    }
}
EOF


验证

ls ~/my-java-app/src/main/java/hello/HelloWorld.java
3.添加 Maven 构建配置
操作目的：定义项目依赖和编译配置，确保 Maven 能正确构建应用。
<maven.compiler.source/target>：指定 Java 版本为 8。


cat <<EOF > ~/my-java-app/pom.xml
<?xml version="1.0" encoding="UTF-8"?>
<project>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.example</groupId>
    <artifactId>my-java-app</artifactId>
    <version>1.0-SNAPSHOT</version>
    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
    </properties>
</project>
EOF

验证 

cat ~/my-java-app/pom.xml | grep '<artifactId>my-java-app</artifactId>'

4.构建 Java 应用
操作目的：编译 Java 代码为 .class 文件，生成可执行程序。
mvn clean：清理旧的构建文件。
mvn package：编译代码并打包（默认生成 target/classes）。


常见问题：
错误：mvn: command not found
原因：未安装 Maven。
解决：运行 sudo yum install maven。

cd ~/my-java-app && mvn clean package

验证

ls target/classes/hello/HelloWorld.class

5.创建 Dockerfile
操作目的：定义 Docker 镜像构建规则，将应用打包为容器。
FROM openjdk:8-jre：基于 OpenJDK 8 JRE 镜像（轻量级运行环境）。
WORKDIR /app：设置容器内工作目录。
COPY target/classes /app/classes：复制编译后的类文件到镜像
CMD [...]：容器启动时执行的命令。


cat <<EOF > ~/my-java-app/Dockerfile
FROM openjdk:8-jre
WORKDIR /app
COPY target/classes /app/classes
CMD ["java", "-cp", "/app/classes", "hello.HelloWorld"]
EOF

验证

cat ~/my-java-app/Dockerfile | grep 'hello.HelloWorld'

6.构建 Docker 镜像
操作目的：将应用及其依赖打包为 Docker 镜像，便于分发和运行。
-t my-java-app：为镜像指定名称和标签（默认为 latest）。
.：使用当前目录下的 Dockerfile。


cd ~/my-java-app && docker build -t my-java-app .

验证

docker images | grep my-java-app
7.运行容器
操作目的：启动容器并验证应用功能。
--name myapp：为容器指定名称（方便后续管理）。

docker run --name myapp my-java-app
验证

docker logs myapp | grep 'Hello, Container World!'

