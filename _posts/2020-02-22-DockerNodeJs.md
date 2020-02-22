# docker上部署nodejs项目
其实node.js官方已经给了一个[部署方案](https://nodejs.org/zh-cn/docs/guides/nodejs-docker-webapp/)。然而，是需要做一些适配的。简单来说就是获取源，然后按照必要的依赖包，之后用node.js启动运行脚本。示例里面就是用express来运行的。
## 设定镜像源
由于国内连接外网太慢，一般要设定镜像源。使用了之后发现阿里的挺快的。怎么配置可以查看[windows版docker镜像源配置](https://jingyan.baidu.com/article/1876c8525ad73c890a137673.html)
## 编写dockerfile脚本
一般来说如果在docker部署的时候做一系列的javascript编译工作有些多余，完全可以在部署前面生成好，然后直接部署dist文件。因此假定有了以下的部署脚本
```dockerfile
FROM node:10
WORKDIR /usr/src/app
ADD main.js main.js
ADD package.json package.json
ADD dist dist
EXPOSE 8080
RUN npm install
CMD [ "node", "main.js" ]
```
main.js是基于express的启动文件，需要复制。而package.json里面实际需要填写的是启动所需的express库，如果都把他们拷贝过去好像很花时间。dist文件夹里面的就是编译好的js/html/image文件了。这样就可以部署成功了。
