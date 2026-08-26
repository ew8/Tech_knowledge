**Deepseek harness游玩记录**
这玩应现在太火了，本着跟风原则先搞一个玩玩看，虽然还没搞清楚能做什么。
**安装**
本着能容器就容器的原则，启动了个通用容器，里面包含node.js的运行环境（必须）。之后根据官网的命令：
npx @deepseek-ai/dsh web （临时启动）  
npm install -g @deepseek-ai/dsh （完整安装）
dsh web(独立启动命令）

安装没啥时候的 到最后出现：
sh web: http://127.0.0.1:3080
dsh web: opening the default browser; pass --no-open to disable
即宣告结束。

**服务器端口映射**
第一个坑，我内网服务器访问方法是： 
容器映射一个端口到宿主机，之后我的PC直接访问服务器的端口，本质是没啥问题的。
但是harness为了安全考虑（很负责），他默认只监听127.0.0.1。一开始以为可以通过一些参数修改。比如--port 修改了默认端口，之后deepseek AI跟我说，你可以试试Truest_HOST还是allow_host的 结果不生效。
之后我去问qwen反倒是给了我正确答案 用--host 果然敌对分子研究的更透彻。 但虽然答案是对的，harness从代码层组织了这个行为：
[root@practical_bohr .dsh]dsh web --host 0.0.0.0 --port 30000
error: --host 0.0.0.0 is intentionally not supported yet for safety: it would expose remote code execution to the network; use 127.0.0.1 instead

这个行为还是很负责的，但是对我就是折磨了。最后想到了用第三方软件监听转发的模式， 下载了一个socat来作为快速流量转发：
socat TCP-LISTEN:30001,fork,reuseaddr,bind=0.0.0.0 TCP:127.0.0.1:30000

这时候30001就是全量监听了，从我的电脑就可以正常访问了:
<img width="1961" height="874" alt="image" src="https://github.com/user-attachments/assets/adf7a10b-70d1-4a60-8054-a2d047f04193" />

TBD
