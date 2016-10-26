# Rxjava使用入门

标签： rxjava 入门
Github上Rxjava的自我介绍：一个在 Java VM上使用可观测的序列来组成异步的、基于事件的程序的库。说的再简单点：简洁的异步和清晰的流程
github地址：[Rxjava](https://github.com/ReactiveX/RxJava)

---

###配置

Module的build.gradle中添加如下配置：

    compile 'io.reactivex:rxjava:1.2.1'
    compile 'io.reactivex:rxandroid:1.2.1'

rxandroid这个主要是一些和Android线程相关的内容，好让rxjava在android中畅行无阻。

---
###三段式
这个是我个人的一种叫法，试着用来理解吧。

    范式：Observable.xxx.subscribe(subscriber/observer)
第一段：Observable起手式，这里就相当于明确了最开始的对象；
第二段：xxx方法（第二段）中间过程，有一大堆供选择方法；
第三段：subscribe()整体落实，启动并完成整个链的执行。
**注意：**有时候，第二段是可以省略的，但是第三段不可以省略因为第三段是建立关系的关键，在特殊情况下，第三部分是可以简化的，就是subscribe()括号内不添加任何代码。

>示例：

    Observable.just("人之初")//第一段
                    .map(new Func1<String, String>() {
                        @Override
                        public String call(String s) {
                            return s+",性本善";
                        }
                    })//第二段
                    .subscribe(new Observer<String>() {//第三段
                        @Override
                        public void onCompleted() {
                            System.out.println("complated");
                        }
                        @Override
                        public void onError(Throwable e) {
                            System.out.println("error");
                        }
                        @Override
                        public void onNext(String s) {
                            System.out.println(s);
                        }
                    });

---
### 异步
用了rxjava以后，我已经不记得AsyncTask/Handler/Runnable的具体代码了，反正是需要很多，而且层级多一点，看起来会更加杂乱。

Rxjava的一部就简单多了，只需要适当的时候简单的指定一下线程即可。不过Rx的代码量上，看起来不是很乐观，但是逻辑上就简洁了很多。

如果说使用异步的程序的话，最典型的就是网络请求了，其他的怎么写我就不举例了（忘记怎么写了，罪过），这里就举例个Rxjava的好了：
>Rxjava拟网络请求过程

    Observable.create(new Observable.OnSubscribe<String>() {
                @Override
                public void call(Subscriber<? super String> subscriber) {
                    String result = ApiUtil.login("admin","123456");
                    subscriber.onNext(result);//将结果传递下去
                }
            })//第一段
                    .subscribeOn(Schedulers.io())//网络获取指定到io线程  第二段的一部分
                    .observeOn(AndroidSchedulers.mainThread())//最后的结果指定会主线程  第二段的一部分
                    .subscribe(new Observer<String>() {//第三段，处理请求结果
                        @Override
                        public void onCompleted() {
                            Log.i("TAG","complete");
                        }
                        @Override
                        public void onError(Throwable e) {
                            Log.i("TAG","error");
                        }
                        @Override
                        public void onNext(String s) {
                            //省略成功后的代码
                        }
                    });

### 流程
记得我小时候，夏天的中午，通常都是跟我妈拿5毛钱，然后去小卖铺买一袋冰棍（袋里有好多小冰棍），然后拿回家，我妈拆开再给我们兄妹三个人分着吃。

就上面我说的这件事，如果用常规的java代码来写的话，大概是这样的：

    void chixuegao(){
        User me = new User();
        me.setMoney(getMoneyFromMom(me));
        goToShop(me);
        me.setBinggun(getBinggun(me.getMoney()));
        ....
    }
    
    int getMoneyFromMom(User u){
        return 5;
    }
    
    void goToShop(User u){
        //省略一堆
    }

Rxjava是这样：

        Observable.just(me)
                .flatMap(me.setMoney(getMoneyFormMom()))//先拿到钱
                .map(gotoShop(me))//去小卖部
                .map(me.setBinggun(getBinggun(me.getMoney())))//买冰棍
                ....//其他步骤
                .subscribeOn(Schedulers.io())//耗时指定到io线程  第二段的一部分
                .observeOn(AndroidSchedulers.mainThread())//吃雪糕指定会主线程  第二段的一部分
                .subscribe(chibinggun());
                
两部分的代码比较（当然，我象征性的写了几句代码），上面的代码更凌乱一些（如果不封装，堆在一起更加凌乱），而Rx通过一步步map()或者flatMap()将整个的流程更加清晰化，也就是逻辑的简洁化。

前面说到了封装，如果对封装不是很有把握，那么在使用Rxjava的时候，会"不得不"对整个事件进行逻辑上的"分化"来代替封装，同样能达到逻辑上的简洁。

代码写到这，我突然想，是不是Rxjava给了原本的"面相对象"一种"面相流程"的更多可能呢？
