# 一致性hash

## 1.hash算法

那么什么是hash算法呢，百度百科的定义如下：

[哈希算法](https://www.zhihu.com/search?q=哈希算法&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})将任意长度的二进制值映射为较短的固定长度的二进制值，这个小的二进制值称为[哈希值](https://www.zhihu.com/search?q=哈希值&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})。哈希值是一段数据唯一且极其紧凑的数值表示形式。

**普通的hash算法在分布式应用中的不足：**

比如，在分布式的存储系统中，要将数据存储到具体的节点上，如果我们采用普通的hash算法进行路由，将数据映射到具体的节点上，如key%N，key是数据的key，N是机器节点数，如果有一个机器加入或退出这个集群，则所有的数据映射都无效了，如果是持久化存储则要做数据迁移，如果是[分布式缓存](https://www.zhihu.com/search?q=分布式缓存&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})，则其他缓存就失效了。

接下来我们来了解，一致性hash算法是怎么解决这个问题的。

## 2.[一致性hash算法](https://www.zhihu.com/search?q=一致性hash算法&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})

一致性哈希提出了在动态变化的Cache环境中，哈希算法应该满足的4个适应条件(from 百度百科)：

### 均衡性(Balance)

平衡性是指哈希的结果能够尽可能分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。很多哈希算法都能够满足这一条件。

### 单调性(Monotonicity)

单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲区加入到系统中，那么哈希的结果应能够保证原有已分配的内容可以被映射到新的[缓冲区](https://www.zhihu.com/search?q=缓冲区&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})中去，而不会被映射到旧的缓冲集合中的其他缓冲区。（这段翻译信息有负面价值的，当缓冲区大小变化时一致性哈希(Consistent hashing)尽量保护已分配的内容不会被重新映射到新缓冲区。）

### [分散性](https://www.zhihu.com/search?q=分散性&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})(Spread)

在分布式环境中，终端有可能看不到所有的缓冲，而是只能看到其中的一部分。当终端希望通过哈希过程将内容映射到缓冲上时，由于不同终端所见的缓冲范围有可能不同，从而导致哈希的结果不一致，最终的结果是相同的内容被不同的终端映射到不同的缓冲区中。这种情况显然是应该避免的，因为它导致相同内容被存储到不同缓冲中去，降低了系统存储的效率。分散性的定义就是上述情况发生的严重程度。好的哈希算法应能够尽量避免不一致的情况发生，也就是尽量降低分散性。

### 负载(Load)

负载问题实际上是从另一个角度看待分散性问题。既然不同的终端可能将相同的内容映射到不同的缓冲区中，那么对于一个特定的缓冲区而言，也可能被不同的用户映射为不同的内容。与分散性一样，这种情况也是应当避免的，因此好的哈希算法应能够尽量降低缓冲的负荷。

接下来说一下具体的设计：

### 2.1[环形hash空间](https://www.zhihu.com/search?q=环形hash空间&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})

按照常用的hash算法来将对应的key哈希到一个具有2^32次方个节点的空间中，即0 ~ (2^32)-1的数字空间中。现在我们可以将这些数字头尾相连，想象成一个闭合的环形。

**NOTE:**当然，节点的个数可以自定义，整个hash环我们可以用TreeMap来实现，因为treeMap是排序的，我们刚好可以利用上。



![img](https://pic2.zhimg.com/80/v2-0a21bff27b5f037748292aa338965d65_720w.jpg)



### 2.2[映射服务器](https://www.zhihu.com/search?q=映射服务器&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})节点

将各个服务器使用Hash进行一个哈希，具体可以选择服务器的ip或唯一主机名作为关键字进行哈希，这样每台机器就能确定其在哈希环上的位置。假设我们将四台服务器使用ip地址哈希后在环空间的位置如下：



![img](https://pic4.zhimg.com/80/v2-252cc4ed5bbb07e5e1e3b27c5eda0d23_720w.jpg)

### 2.3映射数据

现在我们将objectA、objectB、objectC、objectD四个对象通过特定的[Hash函数](https://www.zhihu.com/search?q=Hash函数&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})计算出对应的key值，然后散列到Hash环上,然后从数据所在位置沿环顺时针“行走”，第一台遇到的服务器就是其应该定位到的服务器。



![img](https://pic3.zhimg.com/80/v2-0fb33fe30c7a05eee2abe3784a42f98a_720w.jpg)



### 2.4服务器的删除与添加

2.4.1如果此时NodeC宕机了，此时Object A、B、D不会受到影响，只有Object C会重新分配到Node D上面去，而其他数据对象不会发生变化

2.4.2如果在环境中新增一台服务器Node X，通过hash算法将Node X映射到环中，通过按顺时针迁移的规则，那么Object C被迁移到了Node X中，其它对象还保持这原有的存储位置。通过对节点的添加和删除的分析，一致性哈希算法在保持了单调性的同时，还是数据的迁移达到了最小，这样的算法对[分布式集群](https://www.zhihu.com/search?q=分布式集群&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})来说是非常合适的，避免了大量数据迁移，减小了服务器的的压力。



![img](https://pic4.zhimg.com/80/v2-bf7daae4aa145478dd55fc339ee57ec7_720w.jpg)



### 2.5.虚拟节点

到目前为止一致性hash也可以算做完成了，但是有一个问题还需要解决，那就是**平衡性**。从下图我们可以看出，当服务器节点比较少的时候，会出现一个问题，就是此时必然造成大量数据集中到一个节点上面，极少数数据集中到另外的节点上面。



![img](https://pic4.zhimg.com/80/v2-0ce62cf40bcc5f980cafe285dafe0633_720w.jpg)



为了解决这种数据倾斜问题，一致性哈希算法引入了[虚拟节点机制](https://www.zhihu.com/search?q=虚拟节点机制&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})，即对每一个服务节点计算多个哈希，每个计算结果位置都放置一个此服务节点，称为虚拟节点。具体做法可以先确定每个[物理节点](https://www.zhihu.com/search?q=物理节点&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})关联的虚拟节点数量，然后在ip或者主机名后面增加编号。例如上面的情况，可以为每台服务器计算三个虚拟节点，于是可以分别计算 “Node A#1”、“Node A#2”、“Node A#3”、“Node B#1”、“Node B#2”、“Node B#3”的哈希值，于是形成六个虚拟节点：



![img](https://pic1.zhimg.com/80/v2-5d9cdea01cb4b44162aa41980345e8ac_720w.jpg)



同时[数据定位算法](https://www.zhihu.com/search?q=数据定位算法&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra={"sourceType"%3A"article"%2C"sourceId"%3A98030096})不变，只是多了一步虚拟节点到实际节点的映射，例如定位到“Node A#1”、“Node A#2”、“Node A#3”三个虚拟节点的数据均定位到Node A上。这样就解决了服务节点少时数据倾斜的问题。每个物理节点关联的虚拟节点数量就根据具体的生产环境情况在确定。
