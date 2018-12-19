# mayi_vue
- 仿支付宝蚂蚁基金
- Wetouch解决方案 https://www.wetouch.net/
- 可一键生成Android & IOS客户端
- 可打包成H5项目 & 微信小程序项目

## 关于本地ios设备运行与在线云打包的原理猜测与破解
- 本地ios设备运行原理猜测与破解
  安装了WeTouch的vscode插件后
  插件源码生成在C:\Users\xn067391\.vscode\extensions\uileader.vstouchui-1.5.2
  其中的node_modules\apploader\temp\download\apploader文件夹下即为原始的app壳子
  安装打开后显示 AppLoader 等待传输文件
  在WeTouch工程中右键出现的 WeTouch单独编译当前项目实际生成编译目录C:\Users\xn067391\.vscode\extensions\uileader.vstouchui-1.5.2\node_modules\touchui-pack\dist中
  但是要想在app中正确同步资源，其中的index.html中缺少一段内容 <script src="__app__.js"></script>
  这段内容实际上是在WeTouch工程中右键出现的 WeTouch在ios设备运行 时注入到www文件的index.html中
  在WeTouch工程中右键出现的 WeTouch在ios设备运行 实际上是传输文件到原始的app壳子的/Documents/Pandora/apps/apploader/www文件夹中
  而这个www文件夹是WeTouch工程中右键出现的 WeTouch在ios设备运行 生成的，目录在uileader.vstouchui-1.5.2\node_modules\apploader\temp\www
  那么我们可以用itools等ios手机助手工具将单独编译生成的www文件拷贝替换到原始的app壳子的/Documents/Pandora/apps/apploader/www文件夹中
  然后重启app即可生效

  当然你可以只编译当前项目，然后手动在touchui-pack\dist的index.html添加内容 <script src="__app__.js"></script>，然后复制替换到/Documents/Pandora/apps/apploader/www中

  但是不推荐这么做，最合理的是不连接手机，然后先WeTouch单独编译当前项目 ，再WeTouch在ios设备运行，就会生成最新的资源包了

  还有一种方法是Wetouch导出资源升级包，先单独编译当前项目，然后导出资源升级包（app.wgt）,改成zip解压即为最新的资源包内容


- 在线云打包的原理猜测
  云端重新生成自定义的app壳子，然后同步资源，无非就是改了点app名字和图标，但免费版有流量限制，好像是100M就不让玩了
  此时就需要自己反编译原始app壳子，改改图标和名字，就变成属于自己的app了
  
  ipa反编译
  
  但是每次修改后都需要重签名ipa包 参考 http://www.cocoachina.com/ios/20180530/23571.html?utm_source=tuicool&utm_medium=referral
  自测用过第三种方法iOS App Signer生效 项目地址 https://github.com/DanTheMan827/ios-app-signer
  将项目下载到本地，用Xcode打开iOS App Signer.xcodeproj然后运行
  重签名需要证书和描述文件，个人免费appid开发者可以使用软件制作证书和描述文件添加3台设备（须知道设备udid，且有效期只有7天）
  软件地址 http://www.applicationloader.net/appuploader/download.php
  使用教程 http://www.applicationloader.net/blog/zh/1073.html
  个人免费appid账号可关注公众号--更不更新全看心情--获取
  成功重签名后会生成新的ipa包，最后只能通过爱思助手电脑版安装，itools助手实测无效
  若想将ipa包上传到蒲公英方便添加了udid设备的人安装，则目测只能用99美刀的开发者账号生成证书和描述文件重签名了
  
  apk反编译
  
  使用apktools 软件地址 https://pan.baidu.com/s/1CTi1KiC505gM_k4XaX2_SQ
  将apploader.apk与apktool.bat,apktool.jar放在同一目录
  执行命令 apktool d apploader.apk 将会生成反编译后的apploader文件夹
  在apploader/data文件夹中新建apps/apploader/www文件夹
  将wgt资源包解压后的内容放入到apps/apploader/www文件夹
  修改apk名字 在apploader\res\values\strings.xml中修改为<string name="app_name">苟哥宝</string>
  修改apk图标和启动图标，搜索icon 和splash,替换为对应大小的png图片，可以使用tinypng进行压缩后再替换
  执行命令 apktool.bat b apploader 将会在apploader文件夹中生成重新打包后的dist/apploader.apk
  登录 http://www.uileader.com/uileader/main.do ，在WeTouch / 安卓证书生成 页面中生成keystore证书 记住别名 例如GGB
  将证书重命名为GGB.keystore,与重新打包后的apk放在同级目录中
  执行命令 jarsigner -keystore GGB.keystore apploader.apk GGB 重新签名
  执行命令 jarsigner -verify -verbose apploader.apk 验证签名
  参考 https://blog.csdn.net/sxk874890728/article/details/80486223

- 热更新原理
  无需重新打包签名，Wetouch导出资源升级包后，使用app.ui中的代码安装，参考 http://www.wetouch.net/wetouch_doc/expand/hotUpdate/examples
  可配合 http://www.mockhttp.cn/ 作为模拟简单的版本控制后台接口，当需要更新时更改-version-字段和对应的wgt更新包地址，wgt包放在github.io或内网穿透的服务器上
  每次Wetouch导出资源升级包前都必须修改根目录下的 build_config/config.json 中的versionName和versionCode字段,并且必须大于上一次版本号，并且导出wgt资源包的时候需要指定对应的版本号命名（例如：app1.1.8.wgt），否则热更新时一直处在正在安装状态

## 遇到的问题
- 转小程序后需要重新在小程序项目中配置iconfont相关文件 http://www.wetouch.net/touchwx_doc/transform/transform/conversions 手动调整
    如果配置过一次后，未再更改iconfont库的话可以将小程序项目的static/styles/icon.less备份替换用于任意Touch WX项目
- tab-bar组件配置时为了适配小程序端，不能使用iconfont图标，必须使用本地图片
- Touch WX中：无法在css中引入本地图片，可以使用base64图片，或者线上图片。不可以通过require引入。
- text组件样式设置 font-weight:bold 貌似不生效
- Touch UI中自定义组件是在根目录下创建components文件夹，然后新建自定义组件ui文件
    然后在调用的文件中引入 import MayiCell from '#/component/mayi-cell.ui' ，并注册 components:{MayiCell}
  但自动转换小程序后无用，需要重新修改如下：
  Touch WX中自定义组件是在根目录下创建packages文件夹，然后新建自定义组件wxc文件
    然后在调用的文件中注释掉 import MayiCell from '#/component/mayi-cell.ui' ，注释掉注册 components:{MayiCell}
    接着在config里添加 usingComponents: {'mayi-cell': '../packages/mayi-cell.wxc'}
    如果转换过小程序并生成过小程序项目，需要备份packages文件夹，否则下次转换会丢失
  需要额外注意的是：
    Touch WX中自定义组件中不能使用ui-text等Touch UI组件，只能使用Touch WX组件和小程序原生组件
    Touch WX中自定义组件中如果要使用.mix-flex-center()等less函数库
        1）在packages目录下建一个文件夹styles，然后在styles下建一个mixins.less
        2）mixins.less文件的内容和工程下的static/utils/mixins.less文件内容相同
        3）在组件中引入mixins.less文件
- 由于框架全局设置了行高为1.6倍，所以icon默认行高也是1.6倍。当icon尺寸比较大时，上下留白比较明显。这时可以手动为这个组件设置行高，以保证留白正常。
- flex row布局时若想要某个元素放在最右边，使用 justify-self: flex-end 无效；此时需要对该元素 margin-left: auto 即可
- 自定义组件less使用嵌套语法时，必须保证根元素class属性有且只有一个
- 移动端在线加载pdf方案（用于app端不支持a链接跳转加载pdf） https://mozilla.github.io/pdf.js/web/viewer.html?file=https://chrisgou.github.io/file/201712.pdf
