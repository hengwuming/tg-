博客原地址https://haoduck.com/677.html 本人仅搬运
可以自己尝试编译，不过我比较建议用Docker，这样的话不会有那么多编译环境的问题

我这里演示从Docker运行telegram-cli容器开始

Docker运行Telegram-cli

docker pull peng4740/telegram-cli

docker run -itd -v ~/telegram-cli/:/home/telegramd/.telegram-cli/ --name=tg --restart=always peng（如果没用请通过该命令查找docker ps -a容器id，使用docker strart 容器id 命令启动容器）

4740/telegram-cli

单账号配置
docker exec -it tg telegram-cli

给文件夹权限，让容器里的非ROOT用户能读取配置

chmod -R 777 ~/telegram-cli/

然后按提示输入手机号(带区号，比如+8613800138000)

接验证码(验证码是在Telegram里收的，不是短信)

然后就登录成功了

键入Ctrl + P Ctrl +Q使容器后台运行

多账号配置

写下面配置内容到文件~/telegram-cli/config

这里的示例是3个账号，如果你有更多，就依此类推

default_profile = "profile_1";
 
profile_1 = {
config_directory = ".telegram-cli/profile_1";
msg_num = true;
};
 
profile_2 = {
config_directory = ".telegram-cli/profile_2";
msg_num = true;
};
 
profile_3 = {
config_directory = ".telegram-cli/profile_3";
msg_num = true;
};
写完配置文件需要创建这几个用户的文件夹
也就是这些文件夹：.telegram-cli/profile_1、.telegram-cli/profile_2、.telegram-cli/profile_3
但要注意，这些文件夹对应宿主机的是：~/telegram-cli/profile_1、~/telegram-cli/profile_2、~/telegram-cli/profile_3
创建文件夹的命令应该是mkdir ~/telegram-cli/profile_1，账号不多的话手动敲几行创建一下

很多的话我提供一个脚本，不过应该没有人这么多账号吧
修改下面的dirs=你的用户个数

dirs=10
for i in `seq 1 $dirs`; do mkdir ~/telegram-cli/profile_$i; done
给文件夹权限，让容器的非ROOT用户能读取配置

chmod -R 777 ~/telegram-cli/
进容器登录账号
docker exec -it tg telegram-cli -p profile_1
#输入手机号，TG APP里接验证码登录
#登录完成后直接CTRL+C或者CTRL+P+CTRL+Q退出
#然后依次登录别的账号，即把上面命令最后的profile_1改为profile_2、profile_3，以此类推

使用telegram-cli发消息
docker exec tg telegram-cli -p profile_1 -We "msg <对方名称> <消息>"
注意是名称，而不是用户名或者频道的ID，也就是聊天时显示的名称。当名称有空格时，用下划线代替，嗷嗷翻了issues好久才得到解决方法。

比如

docker exec tg telegram-cli -p profile_1 -We "msg 好鸭机器人 好鸭博客666"

定时发送消息
写一个单账号的发消息比较简便的脚本

cat << EOF > /usr/bin/tg_msg.sh
if [[ ! -z $3 ]];then
/usr/bin/docker exec tg telegram-cli -p $1 -We "msg $2 $3"
else
/usr/bin/docker exec tg telegram-cli -We "msg $1 $2"
fi
EOF
chmod +x /usr/bin/tg_msg.sh

单账号
将命令/usr/bin/tg_msg.sh 好鸭机器人 好鸭博客666" 加入到Crontab
好鸭机器人是对方的名称
好鸭博客666是要发送的消息
记得改成你需要发送的用户和你要发的消息哦

crontab -e
到最后新增一行写入
30 0 * * * /usr/bin/tg_msg.sh 好鸭机器人 好鸭博客666

chmod +x /usr/bin/tg_msg.sh

如果不需要保留crontab日志
30 0 * * * /usr/bin/tg_msg.sh 好鸭机器人 好鸭博客666  > /dev/null 2>&1

多账号
将命令/usr/bin/tg_msg.sh profile_1 好鸭机器人 好鸭博客666" 加入到Crontab
profile_1是Telegram-cli的用户1,还有profile_2,profile_3(前面配置写的)就是用户2,用户3
好鸭机器人是对方的名称
好鸭博客666是要发送的消息
记得改成你需要发送的用户和你要发的消息哦

crontab -e
在最后新增一行写入下面的内容

30 0 * * * /usr/bin/tg_msg.sh profile_1 好鸭机器人 好鸭博客666
如果不需要保留crontab日志

30 0 * * * /usr/bin/tg_msg.sh profile_1 好鸭机器人 好鸭博客666 > /dev/null 2>&1

关于crontab表达式
30 0 * * *表示每天的00:30
30 0,12 * * *表示每天的00:30和12:30
https://tool.lu/crontab/ 可参考该工具生成
按需选择，也可以自己写

选择每天的00:30和12:30可以减低签到没成功的可能
