## 基于html5<audio>的网页音乐播放器

目前只是上传了一个组件，下载后自行导入组件即可使用

> 在开发个人博客的时候想到有个背景音乐听，就写了这个播放器，现博客已经全面开放。
> 地址： [https://www.166zx.com](https://www.166zx.com "张啸的个人博客")

# 在这之前

因为做之前也没多想，没去看看有没有现成的vue插件，所以自己就写了，因为涉及到dom操作，所以把需要的dom操作都封装在一个Music对象当中，看功能之前，先了解一下w3c给出的一些操作api(本次所使用的)

### HTML5 Audio/Video 方法

**methods:**

1. load()  重新加载音频/视频元素
2. play()  开始播放音频/视频
3. pause() 暂停当前播放的音频/视频

**attribute:**

1. duration 返回当前音频/视频的长度（以秒计）
2. ended 返回音频/视频的播放是否已结束
3. currentTime 设置或返回音频/视频中的当前播放位置（以秒计）
5. src 设置或返回音频/视频元素的当前来源
6. volume 设置或返回音频/视频的音量

**event**

- timeupdate 当目前的播放位置已更改时

#music对象

注释应该写的很明确了，一看便懂，此处的dom对象也可自己传值，来降低耦合度，以为一个网页通常只有一个音乐播放器，所以这里偷懒把audio对象写在music对象中了。
	
	//音乐对象
	function Music() {
	  var audio = document.getElementById("audio");
	  //设置音量
	  this.setVoice = function(voice) {
	    audio.volume = voice / 100;
	  }
	  //设置播放进度
	  this.setLength = function(length, duration) {
	    audio.currentTime = length * duration / 100;
	  }
	  //修改播放模式
	  this.modifyModel = function(model) {
	    window.localStorage.setItem("model", model);
	  }
	  //播放和暂停
	  this.setToggle = function(boolean) {
	    if (boolean) {
	      audio.play();
	    } else {
	      audio.pause();
	    }
	  }
	  //更新播放列表
	  this.modify = function(list) {
	    window.localStorage.setItem("list", JSON.stringify(list));
	  }
	  //获取音乐列表
	  this.getList = function() {
	    var list = JSON.parse(window.localStorage.getItem("list") || '[]');
	    return list;
	  }
	  //获取播放模式
	  this.getModel = function() {
	    var model = JSON.parse(window.localStorage.getItem("model") || 1);
	    return model;
	  }
	  //重新播放
	  this.reload = function() {
	    audio.load();
	  }
	}

#功能要点

vue实例监听变化的数据，再通过music对象进行具体的dom操作。

- 播放/暂停，上一首，下一首

播放/暂停：

> 通过监听一个开关，这里叫做**audioToggle**，去布尔值进行播放与暂停的判断

上/下一首：

> 通过一个play()函数，执行播放，通过传入的参数(随机播放，列表播放，单曲循环)判断**播放模式**

> 单曲循环和随机播放比较容易，不需要判断方向，即是上一首还是下一首

> 列表循环需要判断一下**上一首还是下一首**，具体代码如下：

	play(e) {
	  var vm = this;
	  var music = new Music();
	  switch (vm.modelImg) {
	    case 1: //单曲循环
	      music.reload();
	      break;
	    case 2: //列表循环
	      var list = vm.onlineLists;
	      if (list.length > 0) { //判断是否有缓存列表
	        //判断当前音乐在列表中的位置
	        for (var x = 0; x < list.length; x++) {
	          if (list[x].FileHash == vm.musicHash) {
	            if (!e) { //往后下一首
	              if (x !== list.length - 1) { //判断是否是列表最后一首
	                vm.playOnline(list[x + 1])
	              } else { //返回第一首
	                vm.playOnline(list[0])
	              }
	            } else { //往前下一首
	              if (x == 0) { //判断是否是列表第一首
	                vm.playOnline(list[list.length - 1]) //返回列表最后一首
	              } else {
	                vm.playOnline(list[x - 1])
	              }
	            }
	            return;
	          }
	        }
	      } else {
	        music.reload();
	      }
	      break;
	    case 3: //随机播放
	      var list = vm.onlineLists;
	      if (list.length > 0) { //判断是否有缓存列表
	        var num = Math.floor(Math.random() * list.length);
	        vm.playOnline(list[num])
	      } else {
	        music.reload();
	      }
	      break;
	  }
	}



- 实时进度

这里采用监听html5的一个audio事件**timeupdate**来实时返回播放的进度，进度条使用input新属性range，通过v-model的双向绑定的特点，就可以实时修改range的播放进度

	<input type="range" id="rangeLength" v-model="length" min="0" max="100" @input="changeLength()">

- 播放时长

进度和时长的方法类似，就不赘述了，这里修改一个时间属性duration即可。

- 声音调控

同实时进度一样，同样是修改range的值

- 播放模式

分为三种：单曲循环、列表循环、随机播放
播放模式涉及到播放列表，详见播放列表。

- 播放列表

列表循环以及随机播放时候需要用到列表，我将储存好的列表存放在localstorage中(效仿网易云)，如果没有列表默认播放faded(个人比较喜欢的一首歌，很有旋律感)

## 酷狗音乐接口

下面介绍一些酷狗音乐的接口：(一把熟悉jsonp跨域交互的话都可以很轻易的拿到酷狗的音乐文件地址)

如何拿到地址大家可以去**酷狗音乐在线播放器**打开**开发者工具**查找：[http://web.kugou.com/](http://web.kugou.com/ "酷狗")

两个接口：

1. 获取音乐列表：(用于模糊搜索)

http://songsearch.kugou.com/song_search_v2?_callback=jQuery191034642999175022426_1489023388639&keyword=""&page=1&pagesize=10&userid=-1&clientver=&platform=WebFilter&tag=em&filter=2&iscorrection=1&privilege_filter=0&_=1489023388641"

传入若干参数：

- keyword：要搜索的关键字
- page：页数
- pagesize：每页返回的个数
- _callback:自己的回调函数

这样就可以拿到返回的搜索结果，具体大家去试一试就知道啦

2. 获取单个音乐播放路径等：(用于播放修改播放器的url)

这个接口并不是jsonp的接口，受浏览器同源策略的影响，所以需要用到代理来达到请求的目的，可以用node，也可以用nginx，由于还没有写过代理http请求的文章，大家自行百度。

http://www.kugou.com/yy/index.php?r=play/getdata&hash=歌曲hash值

这里我用了node代理，所以请求地址变为

song/yy/index.php?r=play/getdata&hash=歌曲hash值

# 以上就是这次写音乐盒的所有涉及内容了，由于音乐盒比较简单，主要用于在线播放，所以就不加太多的其他内容啦。

附一张音乐盒的ui图

![效果图](https://www.166zx.com/static/img/music_ui.png)

欢迎访问我的个人博客体验效果：[https://www.166zx.com](https://www.166zx.com "张啸的个人博客") **张啸的个人博客**