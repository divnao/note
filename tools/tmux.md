# tmux

## 1. tmux安装与使用

1.终端复用神器

1. [Tmux教程--打造完美的Linux终端](https://blog.csdn.net/williamyuyuyu/article/details/79283374)
2. [Tmux 速成教程：技巧和调整](https://yq.aliyun.com/articles/87316?spm=5176.10695662.1996646101.searchclickresult.ac9b8dd4l1tkkm)
3. [Tmux常用功能总结](https://segmentfault.com/a/1190000007427965)
4. [Linux终端复用神器-Tmux使用梳理](https://www.cnblogs.com/kevingrace/p/6496899.html)
5. [tmux命令使用总结](http://blog.51cto.com/13683137989/1961188)

2. 常用命令

   `$ tmux list-sessions`

   `$ tmux new -s <new_session_name>`

   `$ tmux attach -t <session_name>`

   ctrl+b  + s, 选择一个会话

3. 注意

   注： 1. 单独运行tmux命令，即开启一个tmux会话 ; 2. 不能在tmux会话里面再新建会话，会报错："sessions should be nested with care, unset $TMUX to force"