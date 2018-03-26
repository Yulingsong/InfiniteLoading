# InfiniteLoading
simple demo about Infinite Loading Data.



​	已经有好久没有更新文章了，最近做的东西也不算多，只是没有时间更新文章，今天继续弄个给小白看的文章，一般的移动端搜索结果页或者那种分页加载的页面需要用的，上拉无限加载数据，不知道这个题目对不对，大家看了就知道是不是自己需要的东西了。

![Mar-26-2018 14-35-30.gif](https://upload-images.jianshu.io/upload_images/1062695-abc8893f93f1e5b4.gif?imageMogr2/auto-orient/strip)

​	下面直接说完成过程吧，这个可以说是非常简单的一个小功能了，对于小白来说也能一眼就能看明白的，当然，要是有更好的方法，希望大家也跟我说下。

##### 第一步：构建页面框架
​	这一步很简单，一般自己的项目都会有自己的设计，我就直接写一个简单粗糙的列表页。html部分是：

    <div class="nav">
      上拉分页无限加载
    </div>
    <!--显示的大概样子-->
    <div class="container">
        <div class="lineItem">1</div>
        <div class="lineItem">2</div>
        <div class="lineItem">3</div>
        <div class="lineItem">4</div>
        <div class="lineItem">5</div>
        <div class="lineItem">6</div>
        <div class="lineItem">7</div>
        <div class="lineItem">8</div>
        <div class="lineItem">9</div>
        <div class="lineItem">10</div>
        <div class="lineItem">11</div>
        <div class="lineItem">12</div>
        <div class="lineItem">13</div>
        <div class="spinner">加载中。。。</div>
        <div class="spinner">- 数据已经加载到底 -</div>
    </div>

css部分是这样的：

    <style>
        *{
            margin: 0;
            padding: 0;
        }
        .container{
            width: 100vw;
            height: calc(100vh - 44px);
        }
        .nav{
            width: 100vw;
            height: 44px;
            line-height: 44px;
            text-align: center;
            font-size: 20px;
        }
        .lineItem{
            width: 100%;
            height: 100px;
            margin-top: 10px;
            background-color: lightgray;
            line-height: 100px;
            text-align: left;
            font-size: 12px;
            overflow: hidden;
        }
        .spinner{
            width: 100%;
            height: 50px;
            text-align: center;
            line-height: 50px;
            font-size: 16px;
            background-color: lightcyan;
            margin-top: 10px;
        }
    
    </style>

写完显示出来的样子是下图：

![1](https://upload-images.jianshu.io/upload_images/1062695-0a043ede02713bd5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![2](https://upload-images.jianshu.io/upload_images/1062695-8a7c07ef3cb75112.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##### 第二步：编写逻辑部分

​	我这边引入了jquery，大家用js写的话，部分地方有点差异，不过都没啥区别。大家注意下，这块我用的ajax请求的url是公司的，所以就拿掉了，大家替换上各自项目的url和请求数据就行。

    //声明部分变量
    var pageNum = 1;//页码
    var pageSize = 10;//每页多少条数据
    var infinite = true;//防止不停调接口加载新数据,false的时候就不能请求接口
    var count = 0;//总数,判断是否已经加载到底了
    var dataCache = [];//数据缓存，根据接口不同情况，可以考虑不用这个。
    //请求方法
    function sendRequest(num){
        var sendData = {
            //其它请求参数
            pageNum:num,
            pageSize:pageSize
        };
        if(infinite == true){
            //只有infinite是true的时候才可以请求接口
            infinite = false;
            $.ajax({
                type: "POST",
                url: url,
                data:sendData,
                dataType: 'json',
                success: function (data) {
                    console.log(data);
                    if(data.code == 200 && data.data){
                        count = data.data.count ? data.data.count:0;
                        if(data.data.products && data.data.products.length != 0){
                            setPage(data.data.products);//单独的拼接html,渲染页面的方法。
                        }else{
                            //这边处理当数据为空的时候,并且页数为第一页的时候,页面应该有相应的显示
                            // 比如暂无数据列表之类的文案,背景之类的。
                            //并且会有种可能就是在加载到部分页数之后,请求回来的数据为空,页面显示也是有相应的变化
                            if(pageNum == 1){
    
                            }else{
    
                            }
                        }
                        //在页面渲染之后,此时的数字加1,为了下一次的加载
                        pageNum = data.data.pageNum + 1;
                    }else{
                        //接口返回部分code不正确的时候,页面应有相应的显示
                        console.log(data.content);
                    }
                    //设置可以继续请求接口。
                    infinite = true;
                },
                error: function (err) {
                    infinite = true;
                    console.log('系统异常');
                }
            });
        }else{
            infinite = true;
        }
    }
    sendRequest(pageNum);
    
    //渲染页面方法
    function setPage(data){
        console.log(data);
        $(".spinner").remove();//移除加载条
        for(var i = 0;i < data.length;i++){
            /*这边也可以做一些缓存数据的操作,记住一些数据,然后进行一些别的操作,比如说切换tab的时候,不刷新页面渲染页面
            也可以根据不同接口返回的数据来做判断是否数据加载到底。我用的接口不支持返回数量,所以就用这种方式来判断*/
            dataCache.push(data[i]);//将数据添加缓存数组中,适合数据量少的时候,不过数据量大的时候也可以用
            //拼接html,渲染页面
            var html = setHtmlModel(data[i]);
            $(".container").append(html);//添加html
        }
        //判断是否加载到底,如果还有数据的话,就直接显示加载中,如果没有数据的话,就直接显示加载到底
        //这部分的样式需要各自项目自己定义,我这边就只做粗糙的显示
        var loadHtml = '';
        if(count > dataCache.length){
            loadHtml = '<div class="spinner">加载中。。。</div>';
    
        }else{
            loadHtml = '<div class="spinner">- 数据已经加载到底 -</div>';
        }
        $(".container").append(loadHtml);
    }
    
    //拼接html方法,这边的html因为太简单,所以这块看起来有点多余,在部分拼接html复杂的情况下,这样写可以看着好看点,修改添加的时候也方便点
    function setHtmlModel(data){
        var html = '';
        html += '<div class="lineItem">'+data.name+'</div>';
        return html;
    }


​	这部分的东西基本上就是请求数据，渲染页面，添加加载框或者加载到底的显示，不过最重要的部分无限加载这块还没有添加上去，所以此时你翻来覆去只能请求一次。


##### 第三步：添加自动加载的方法

    //自动加载新数据,当滚轮滚到离页面下面一段距离的时候,就自动更新数据,请求接口
    $(function(){
        $(window).scroll(function () {
            //下面三个参数很重要,大家可以自行百度下具体的解释,一般简单的无限滚动加载都会用到这个
            var scrollTop = $(this).scrollTop();//匹配元素的滚动条的垂直位置
            var scrollHeight = $(document).height();//匹配元素document的高度
            var windowHeight = $(this).height();//匹配元素的高度
            if(scrollHeight - scrollTop - windowHeight < 200 ){
                //这边多做一个判断,如果还有剩下没有加载的数据就进行加载,如果没有,就不做请求
                if(count > dataCache.length){
                    sendRequest(0,0,pageNum);
                    infinite = false;
                }
            }
        });
    });


​	走完这三步就完成了一个简单的自动加载的列表页了。
说实话，写前端也有段时间了，按道理来说应该写一点复杂点的东西，像这样简单的东西随随便便都能找到，可能也是我学艺不精，还在慢慢学习中，希望以后能给大家带来更好的文章。有什么问题大家也可以跟我一起交流交流。





