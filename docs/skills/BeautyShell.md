
https://ohmyposh.dev/docs/installation/windows

https://ohmyposh.dev/docs/installation/linux

让登录 shell 自动加载 ~/.bashrc：
编辑 ~/.bash_profile（若不存在则创建）：
bash
nano ~/.bash_profile


添加以下内容：
bash
if [ -f ~/.bashrc ]; then
    source ~/.bashrc
fi


保存并退出后，重启终端即可生效。