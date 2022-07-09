## node

1. 安装`brew install node`

2. `node -v` ，`npm -v` 验证安装是否成功

3. `npm config set registry https://registry.npm.taobao.org`

   配置后可通过下面方式来验证是否成功

​       `npm config get registry`

​       在` ~/.npmrc `加入下面内容，可以避免安装 node-sass 失败

​        `sass_binary_site=https://npm.taobao.org/mirrors/node-sass/`

​	   直接使用 vi ~/.npmrc

## python

建立虚拟环境

1、python3 -m venv  文件夹

2、source 文件夹/bin/activate

3、退出：deactivate

sys.stdin使用command+d结束输入

## JS

```js
//用于视频加速
var tags=document.getElementsByTagName("video");
[...tags].forEach(val=>val.playbackRate = 2);
```





