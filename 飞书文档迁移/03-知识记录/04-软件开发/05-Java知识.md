---
created: '2024-04-28 09:08:50'
feishu_url: https://wk5tnvpfe7.feishu.cn/docx/Dx22dBUhEoCxjwxSn3Mc4D2PnFf
modified: '2024-05-15 16:35:16'
source: feishu
title: 05-Java知识
---

05-Java知识
解包打包
解包
jar -xvf test.jar
打包
jar cvfM0 **.jar ./
# 其中./表示当前目录下所有的文件进行打包，切记，指令中M后边必须添加0。-0 仅存储; 不使用任何 ZIP 压缩，这样可以保证使用解压之前的manifest.mf文件。
# https://blog.csdn.net/FelixWang0515/article/details/87924148?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-87924148-blog-118612538.235%5Ev43%5Epc_blog_bottom_relevance_base9&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-3-87924148-blog-118612538.235%5Ev43%5Epc_blog_bottom_relevance_base9&utm_relevant_index=6
Maven构建
增加构建号
<plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <finalName>${project.artifactId}-${project.version}-${build.sid}</finalName>
                    <appendAssemblyId>false</appendAssemblyId>
                    <descriptors>
                        <descriptor>assembly/assembly.xml</descriptor>
                    </descriptors>
                    <archive>
                        <manifest>
                            <mainClass>com.iflytek.aicc.seat.app.AiccSeatApplication</mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>

mvn clean -Dmaven.test.skip=true    install -pl aicc-plat/aicc-plat-core -am

