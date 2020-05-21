Docker环境中使用 Gerrte/Jenkins

Jenkins环境搭建
1.获取 Jenkins
 docker pull jenkins/jenkins
 
2.启动 Jenkins
  java -jar jenkins.war --httpPort=8080

3.登陆并初始化Jenkins
 3.1.在浏览器中输出http://localhost:8080 ，打开Jenkins页面，等待初始化。
 3.2.根据提示输入初始管理员密码，进入初始配置
 3.3.选择安装插件等待安装完成
 
4.安装Gerrit Trigger插件
 4.1.Manage Jenkins-> Manage Plugins->Available
 4.2.搜索Gerrit Trigger
 4.3选择Gerrit Trigger并点击 Download now and install after restart
 4.4等待安装完成并重启

5.添加凭证
 5.1.回到主页
 5.2.点击左侧 Credentials->System-> Global credentials->Add Credentials
 5.3.选择SSH Username with private key
 5.4.填写信息，并拷贝本地私钥信息填入Private Key一栏中
 5.5.Save
 
6.配置Gerrit服务器
 6.1.Manage Jenkins-> Gerrit Trigger -> Add New Server
 6.2.填写服务器名称，Hostname为本机Ip地址，	Frontend URL为Gerrit的URL，SSH port默认，填写Username，填写Email地址，SSH Keyfile为Git的私      钥文件，点击Ok
     GerritTriggerSever.png
 6.3.SSH Key file设置为Gerrit SSH Key对应的私钥文件
 6.4.点击 Test Connection，出现Success即为通信成功
 6.5.点击Sava保存配置

7.添加 Item
 7.1.回到主页，点击New Item
 7.2.填写名称Hello syrius，选择Pipeline
 7.3.在Build triggers中选择Gerrit event，Gerrit Trigger中的Choose a Server选择GerritTriggerSever，Gerrite Project第一个Type选择Plain，pattern填写Hello syrius，第二个Type选择Path，pattern填写××/×
HelloXXXX 表示需要匹配的项目名
Path **/*表示匹配任意分支路进，还可选择其他方式匹配
Plain: The exact name in Gerrit, case sensitive equality.
Path: ANT style pattern. Ex: "**/base/*"
RegExp: Regular expression. The expression should match the whole line, so to use the example with base above, ./base/?.
  7.4.在Pipeline中选择pipeline script from SCM，SCM选择Git，Repository URL为你的仓库地址，这里是ssh://gongzheng@10.42.0.239:29418/Hello%20syrius， 注意仓库中间的localhost应填写为Ip地址，
填写如下信息 Pipeline SCM
注意这里的仓库地址一定不能填写localhost，应为在Jenkins在docker中运行，所以要填写本机ip地址Jenkins才能正确访问Gerrit仓库
URL填写之前在Gerrit上新建的仓库地址
Refspec:+refs/*:refs/remotes/origin/*表示将远程仓库的refs/目录下的文件fetch到本地refs/remotes/origin/目录，这样才能获取Gerrit上所有的提交代码分支，否则Jenkins拉取远程代码时会找不到最新的提交代码
Branch to build */ 表示任意分支
Additional Behaviours要添加Strategy for choosing what to build并设置为Gerrit Trigger. 这样才能正确获取出发分支的commit信息
7.5.保存

Gerrit搭建
1.获取Gerrit
java -jar gerrit*.war init -d ~/gerrittest

2.填写配置信息
配置文件的路径在/home/ken/gerrittest/etc/gerrit.config

3.启动Gerrit
/home/ken/gerrittest/bin/gerrit.sh start

4.登陆Gerrit
浏览器中输入http://localhost:8081进入grrite

5.注册帐号
NewAccount->填写信息->SAVE

6.添加SSH key
setings->SSH key，设置Public Key

7.新建仓库
 7.1.BROWSW->Repositories->CREATE NEW
 7.2.填写仓库名称

8.添加账户到组
进入Browser->Repo->Groups->Non-Interactive Users->Members，将自己的账户添加到SSEmbedded账户组中

9.Acccess权限设置
 9.1.BROWSW->Repositories->Hello syrius->Access->EDIT
 9.2.将用户组添加到相应的权限下。配置Code-Review，Label Verified，Push Merge Commit，Submit的相应权限

10.提交代码及更改
 10.1.注意在使用clone命令（例如git clone "ssh://gongzheng@localhost:29418/Hello%20syrius"）将仓库拉到本地进行修改后commit是不带Changed-Id的。会造成push的时候报错 (missing Change-Idincommit message footer)，需要在clone到本地的命令后增加scp拷贝commit-msg，
 git clone "ssh://gongzheng@localhost:29418/Hello%20syrius && scp -p -P 29418 gongzheng@localhost:hooks/commit-msg project/.git/hooks/
 10.2.提交代码
 git status   
 git diff
 git add --all
 git commit -m "add test.c"
 git push origin HEAD:refs/for/master
 
 11.在仓库中添加JenkinsFile文件
    pipeline {
     agent any
     stages {
         stage('build') {
             steps {
                 echo 'build start'
             }
         }
         stage('deploy') {
             steps {
                 echo 'deploy start'
             }
         }
     }
     post {
         always {
             echo 'One way or another, I have finished'
             deleteDir() /* clean up our workspace */
         }
         success {
           echo 'success'
         }
         failure {
           echo 'failure'
         }
     }
 }
 
 
 测试
 
提交代码，会取法jenkins自动运行，并将结果返回给Gerrit，形成Verified打分




 
