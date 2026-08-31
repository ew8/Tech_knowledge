**免费静态网页解决方案 git page+cloudflare**  

**背景：**  
收到一个需求，有一个固定二维码，扫完后可以下载一个特定的PDF。

**思路**  
听起来很简单，最简单搭建一个apache+公网IP解析就可以。但是学过信息安全就考虑的很多，比如被攻击，被扫描，被开户（有点极端）。再考虑到这个页面以后可能做正经官网，这时候正规流程就成了域名+防御+服务器。本着能省就省的原则，于是想到了github pages,免费的静态页面部署，你只需要把你的代码放在github仓库中他会自动帮你部署并且挂在到你自己的git域名下(这里需要注意到是public的repo部署是免费的，private部署需要付费），但是由于众所周知的原因，git域名国内会概率抽风，这时候就请出大善人cloudflare。免费的域名解析+加速+一些防御。个人小网站免费版足以。最后就是找一个服务商买一个便宜的域名，把他们三个串起来就好了。

二维码的解析参考了[wiloon](https://wiloon.com)的博客:

https://wiloon.com/wechat-qr-pdf-cloudflare-pages/  
这个例子考虑了很多，包括不同浏览器，直接解析PDF可能无法触发预览和下载等问题,基本是开箱即用，本例子主要是研究页面落地发布，所以这里先不考虑这部分代码嵌入问题。

**实现**
首先我们需要先搞一个git仓库，并且启动免费的github page功能：
很简单的index.html +我们的PDF
<img width="2520" height="384" alt="image" src="https://github.com/user-attachments/assets/1859de62-566b-4dc1-81ae-0e81ff8603cf" />

之后点击项目的settings->pages,点击deploy form a branch, 选择好branch和文件夹 之后save。 正常情况过一阵上面绿色的地方会出现一个你的id.github.com的域名,把这个记下来，之后要用：
<img width="1699" height="673" alt="image" src="https://github.com/user-attachments/assets/968f478a-3946-4697-878b-19126ad8e925" />


之后我们需要一个域名，这里选的是阿里云主要是省心，一般来说cxyz/me/vip这种域名+比较长没什么意义的域名会相对便宜很多。 com/cn这种顶级的就明抢了
此图为一个比较简单域名10年的费用
<img width="904" height="554" alt="image" src="https://github.com/user-attachments/assets/cca079c6-663d-4c79-8774-ccb9d6d44179" />

当我换了个单词，拉倒7~8个字母后，价格就下来了很多：  
<img width="890" height="79" alt="image" src="https://github.com/user-attachments/assets/50838dbb-4a40-4322-9f48-386bdd5e31ca" />

买完后，我们需要去我们域名供应商那DNS改成cloudflare的DNS，这一步的目的是阿里云会像顶级DNS跟服务器通知，以后这个域名通过cloudflare的dns服务器解析，不再从阿里云的DNS解析:

<img width="2560" height="569" alt="image" src="https://github.com/user-attachments/assets/c6aa0011-63c6-4590-a1c1-eee84b5d5a67" />

之后我们就可以关闭阿里云了，接下来我们去cloudflare注册个账号，并且把我们的域名绑定进去:
<img width="2265" height="320" alt="image" src="https://github.com/user-attachments/assets/d5cddaa1-3663-40c7-b9f5-ead1ea7d6d06" />
如果是第一次进来这个页面是可以直接添加，不需要点击add我记得。

之后套餐选择免费的足以：
<img width="1724" height="740" alt="image" src="https://github.com/user-attachments/assets/712bf8f9-37c3-43ae-bf5f-34b17409e56e" />

之后我们进入domain的管理界面： 选择DNS->RECORD

<img width="2485" height="1039" alt="image" src="https://github.com/user-attachments/assets/4bb14cb8-2e49-4df9-8f57-774beb99395b" />

这里我们需要添加5条：
首先我们第一步有一个github的域名，我们新建一个cname类型的record，name没有特殊定义务必写www，他会把name作为前缀跟你的域名进行连接，这里我一开始以为单纯是个名字结果导致连接不同，当然你可以自定义
域名来修改这里。 后面的target就是你的github的那个域名
这一步的目的是为了你所有的子域名指路，即当你访问www.host.com及其子路由，DNS会知道我们应该解析到哪里。
<img width="2009" height="275" alt="image" src="https://github.com/user-attachments/assets/10b277c2-02a1-4a4c-9f7a-c616d0253759" />

之后需要添加4条A类型路由，name需要填写你完整的域名，如www.host.com 后面IPV4 address是固定的，指向github的IP地址。  
这一步的目的是让请求者知道下一步应该去哪解析这个域名

<img width="2055" height="485" alt="image" src="https://github.com/user-attachments/assets/92631f87-208f-4352-950d-83eec4b719f5" />


这里讲的比较混乱，我们举个例子：  
假设我们的域名是www.host.com 当我请求后，浏览器会找到你的DNS服务商问这个域名的IP是什么。  
DNS服务商没见过这个域名，他就会去上一层继续问，直到最后到了顶级跟服务器，查到了，哦这个是cloudflare管理的，你去cloudflare问问是哪个IP，他的DNS服务器是xxxx（就是我们在阿里云填写的DNS）  
之后我们来到cloudflare DNS服务器，把域名交给了他，他根据cname解析出了真实域名，即github.io的那部分，之后又通过A类型record，知道了我需要访问哪个IP地址，并且访问他。  
这里我在研究时候有个疑问，因为DNS目的就是域名和IP的解析，我们最后访问的是IP，这里既然我们都有了IP为什么还要配置个CNAME。 后来查了下，github是读取你的访问头，你需要将你的域名放进去，他才会正常解析出你的页面。  
我理解就相当于一层NAT了，公网IP是固定的，通过你的ID映射到真正的内网服务。不过这里并不是真正意义上的NAT，只是举个例子方便理解。  
好了，下一步我们找到了github，可算是最终找到了你的页面服务，最后返给浏览器，结束了这次请求。  

不过这里你会看到cloudflare帮我们做了一次隐藏，包括域名和IP。这样别人攻击你的网站就比较困难找到你的根网站，而且可以防范一定量的DDNS攻击。 如果你把github那部分换成你家自己的域名或者IP，这样就为个人网站用户提供了很大的安全防护作用。

