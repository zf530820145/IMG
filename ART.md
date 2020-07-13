从图1可以看到，ART运行时堆划分为四个空间，分别是Image Space、Zygote Space、Allocation Space和Large Object Space。
- Image Space、Zygote Space、Allocation Space是在地址上连续的空间，称为Continuous Space，
- Large Object Space是一些离散地址的集合，用来分配一些大对象，称为Discontinuous Space。

  > 在Image Space和Zygote Space之间，隔着一段用来映射 system@framework@ boot.art@classes.oat 文件的内存。
    它是由在系统启动类路径中的所有DEX文件翻译得到的

  - Image Space空间就包含了那些需要预加载的系统类对象。
    > 这意味着需要预加载的类对象是在生成 system@framework@ boot.art@classes.oat 这个OAT文件的时候创建并且保存在文件system@framework@ boot.art@classes.dex中，
     以后只要系统启动类路径中的DEX文件不发生变化（即不发生更新升级），那么以后每次系统启动只需要将文件system@framework@ boot.art@classes.dex直接映射到内存即可，
     省去了创建各个类对象的时间。
     之前使用Dalvik虚拟机作为应用程序运行时时，每次系统启动时，都需要为那些预加载的类创建类对象。因此，虽然ART运行时第一次启动时会比较慢，但是以后启动实际上会更快。

     由于 system@framework@ boot.art@classes.dex文件保存的是一些预先创建的对象，并且这些对象之间可能会互相引用，
     因此我们必须保证 system@framework@ boot.art@classes.dex 文件每次加载到内存的地址都是固定的。
     这个固定的地址保存在 system@framework@ boot.art@classes.dex 文件开头的一个Image Header中。
     此外，system@framework@ boot.art@classes.dex 文件也依赖于 system@framework@ boot.art@classes.oat 文件，因此也会将后者固定加载到Image Space的末尾。

  - Zygote Space和Allocation Space与Dalvik虚拟机垃圾收集机制中的Zygote堆和Active堆的作用是一样的。
    - Zygote Space在Zygote进程和应用程序进程之间共享的，而Allocation Space则是每个进程独占的。

    - Zygote进程一开始只有一个Image Space和一个Zygote Space。
      在Zygote进程fork第一个子进程之前，就会把Zygote Space一分为二，原来的已经被使用的那部分堆还叫Zygote Space，而未使用的那部分堆就叫Allocation Space。
      以后的对象都在Allocation Space上分配。

    - 通过上述这种方式，就可以使得Image Space和Zygote Space在Zygote进程和应用程序进程之间进行共享，而Allocation Space就每个进程都独立地拥有一份。注意，虽然Image Space和Zygote Space都是在Zygote进程和应用程序进程之间进行共享，但是前者的对象只创建一次，而后者的对象需要在系统每次启动时根据运行情况都重新创建一遍。
