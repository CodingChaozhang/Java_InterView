# Redis进阶应用篇(Python)

<img src="./imgs_redis/1.jpg">

# 一、使用Redis管理登录令牌

大多数网站都会使用`cookie`记录用户的身份。`cookie`是由少量数据组成的字符串（通过还要经过加密）。网站会要求游览器存储这些数据，并在向服务端发起请求时将这些数据传回给服务端。



通常，用于处理登录(识别用户身份)的`cookie`分为两种：

- `签名式cookie`

  - 存储包含用户`ID`等可直接识别用户的信息
  - 附加一个签名，核对`cookie`信息是否被恶意篡改

- `令牌式cookie`

  - 存储一个随机字符串(令牌)
  - 通过在服务端的数据库中查找随机字符串和用户的对应关系识别用户身份

  | 类型            | 优点                                                         | 缺点                                                   |
  | --------------- | ------------------------------------------------------------ | ------------------------------------------------------ |
  | 签名式 `cookie` | 直接存储用户信息，方便验证用户身份；可以包含额外信息；对 `cookie` 进行签名较简单 | 遗漏签名会导致安全漏洞，加密方法不当会泄露用户敏感信息 |
  | 令牌式 `cookie` | `cookie` 体积小，可加快通信速度                              | 需要使用数据库存储令牌，                               |

## 1.1 如何核对令牌

首先，约定令牌的存储方式，使用一个哈希存储登录令牌与用户的映射关系，其中：

- 哈希键名为 `login`
- 登录令牌作为**域**
- 用户 `ID` 作为**值**

```python
# 核对令牌，并返回该令牌对应的用户 ID
def check_token(token):
    # 请在下面完成要求的功能
    #********* Begin *********#
    return conn.hget('login',token)
    #********* End *********#
```

使用 `hget()` 方法从 `Redis `中取出并返回令牌对应的用户 `ID`，而**当令牌不存在时，该方法则会返回 `None`**，从而达到核对检查的效果。

## 1.2 如何更新令牌

更新令牌需要做两个工作：

- 记录用户令牌
- 记录令牌生成时间

由于令牌存在被人窃取的可能，所以我们不允许令牌永不过期。通过记录令牌生成的时间戳，我们可以通过定期清理的方式，清理掉一定时间前生成（过老）的令牌，从而实现令牌的时限性，在一定程度上也减少了 `Redis `的存储量，避免内存过高消耗。



使用**有序集合**记录令牌生成时间能让我们更便捷的根据时间戳对令牌进行排序，然后再对一定时间前生成（过老）的令牌进行删除。将时间戳作为分值，令牌作为成员，记录到有序集合 `recent:token` 中：

```python
# 更新令牌，同时存储令牌的创建时间
def update_token(token, user_id):
    # 请在下面完成要求的功能
    #********* Begin *********#
    timestamp = time.time()
    pipe = conn.pipeline()
    pipe.hset('login',token,user_id)
    pipe.zadd('recent:token',token,timestamp)
    pipe.execute()

    #********* End *********#
```



## 1.3 如何定期清理无用信息

随着登录用户的增多，令牌存储所需的内存也会不断增加，这时，我们需要定期清理过期的令牌数据。

决定令牌的有效时间需要权衡数据安全与用户体验两方面：

- 有效时间过短，则用户需要频繁的输入账户密码登入系统
- 有效时间过长，则令牌泄露的可能性增大，伪造用户身份的可能性也越大

将令牌的有效时间设置为一个星期（`86400`秒），在每次清理令牌数据时，我们找到令牌生成时间在一个星期前的数据，并将这些令牌和令牌生成时间数据全部删除。



寻找一个星期前生成的令牌是关键，我们可以使用当前 `Unix` 时间减去 `86400` 得到一个星期前的 `Unix` 时间戳，然后使用有序集合命令 `ZRANGEBYSCORE` 获取有序集合 `recent:token` 中所有分值（生成时间）大于等于 `0`，小于等于一个星期前的 `Unix` 时间戳的成员：

```python
# 清理过期令牌
def clean_tokens():
    # 请在下面完成要求的功能
    #********* Begin *********#
    one_week_ago_timestamp = time.time()-86400

    expired_tokens = conn.zrangebyscore('recent:token',0,one_week_ago_timestamp)
    conn.zremrangebyscore('recent:token', 0, one_week_ago_timestamp)
    conn.hdel('login', *expired_tokens)
    #********* End *********#
```



# 二、使用Redis实现购物车

实现购物车的后端处理逻辑

- 存储商品信息
- 存储购物车信息
- 获取购物车信息

## 2.1 存储商品信息

商品包含多个属性，例如：名字，价格，描述等等。使用哈希能够将商品的所有信息存储在一个键 `item:*:info`，其中 `*` 是商品 `ID`，例如，商品 `ID` 为 `1` 的哈希键 `item:1:info` 中的内容为：

```json
{
"name": "儿童棉马甲加厚",
"price": 14.9
}
```



```python
# 添加商品
def add_item(name, price):
    # 请在下面完成要求的功能
    #********* Begin *********#
    item_id = conn.incr('item_id')
    item_info_key = 'item:' + str(item_id)+ ":info"
    conn.hmset(item_info_key,{"name":name,"price":price})
    conn.expire(item_info_key,30 * 24 * 60 * 60)

    return item_id
    #********* End *********#
```



## 2.2 存储购物车信息

购物车的定义十分简单，我们可以将每个用户的购物车都看作是一个哈希，这个哈希存储着商品 `ID` 与加入购物车的数量之间的映射关系，由于购物车与用户相关，所以购物车的哈希键名为 `cart:*`，其中`*`是用户`ID`。



- 当用户将某件商品加入到购物车时
  - 应该将该商品 `ID` 和加入购物车的数量添加到哈希中
  - 如果购物车中已有该商品，则应该根据新的数量更新哈希
- 如果用户将某件商品移出购物车时
  - 应该从哈希中删除该商品 `ID` 对应的域

```python
# 加入购物车
def add_to_cart(user_id, item, count):
    # 请在下面完成要求的功能
    #********* Begin *********#
    if count > 0:
        conn.hset("cart:"+user_id,item,count)
    else:
        conn.hrem('cart:'+user_id,item)
    #********* End *********#
```



```python
# 获取购物车详情
def get_cart_info(user_id):
    # 请在下面完成要求的功能
    #********* Begin *********#
    return conn.hgetall("cart:"+user_id)
    #********* End *********#
```

# 三、使用Redis做页面缓存

在动态生成网页的时候，通常会使用模板（`template`）来简化网页的生成，现在已经不再需要我们手写一整个页面。通常，一个网页包括头部，尾部，侧边栏，工具栏和内容域等部分组成，每个部分都会独立使用一个模板来编写。



我们会通过缓存的方式避免生成这些页面，减少动态生成页面所花费的时间，降低服务器的负载，提高网页访问速度。



我们需要在请求被响应之前，通过一个缓存函数判断：

- 尝试从缓存中取出该请求的响应页面并返回
- 若上述缓存不存在（失效），则：
  - 响应该请求，生成页面
  - 缓存至 `Redis`，生存时间为`10`分钟
  - 将该页面返回

我们可以使用字符串键来存储缓存页面，所以你可以使用 `GET` 命令尝试取出缓存页面，但当我们想要缓存页面时，则应该使用 `SETEX` 命令，该命令和 `SET` 命令的区别是，它是一个原子性（`atomic`）操作，关联值和设置生存时间两个动作会在**同一时间内**完成，所以它在 `Redis `用作缓存时很常用。它的语法如下：

```
conn.setex(key, value, seconds)
```

其中：`seconds` 是键的生存时间，单位为秒。

```python
import redis

conn = redis.Redis()

# 使用 Redis 做页面缓存
def cache_request(request_url):
    # 请在下面完成要求的功能
    #********* Begin *********#
    page_key = 'cache:' + str(hash(request_url))
    content = conn.get(page_key)

    if not content:
        content = 'content for ' +request_url
        conn.setex(page_key,content,600)

    return content


    #********* End *********#
```

# 四、使用Redis做数据缓存

使用 `Redis `做数据缓存的做法是：

- 编写一个将数据加入缓存队列的函数
  - 通过一个有序集合`cache:list`存储数据加入缓存的世界
    - 成员为数据`ID`（唯一标识）
    - 分值为当前时间(`time.time()`)
  - 通过一个有序集合`cache:delay`存储数据更新周期
    - 成员为数据`ID`（唯一标识）
    - 分值为更新周期，单位为秒
- 编写一个定时缓存数据的函数
  - 将数据转换成`json`格式
  - 然后将上述`json`存储到`redis`中
  - 根据缓存更新周期定时更新`redis`中的缓存键

## 4.1 将数据加入缓存队列

有序集合 `cache:list` 作为缓存队列，其需要依赖数据更新周期有序集合 `cache:delay`，加入某数据的更新周期不存在，那么我们则需要删除掉该数据的缓存。在这里，我们采用一种更便捷的方式避免数据缓存和取消数据缓存，那就是将该数据的更新周期设置为小于等于 `0`。

```python
# 将数据加入缓存队列
def add_cache_list(data_id, delay):
    # 请在下面完成要求的功能
    #********* Begin *********#
    conn.zadd('cache:delay',data_id,delay)
    conn.zadd('cache:list',data_id,time.time())
    #********* End *********#
```



## 4.2 缓存数据

我们将数据加入到缓存队列后，就将有序集合 `cache:list` 的分值看作下一次要更新的时间，所以我们可以根据分值对有序集合 `cache:list` 进行排序，并连同分值一起取出从小到大顺序的第一个成员（最可能需要更新的成员）：

```python
# 缓存数据
def cache_data():
    # 请在下面完成要求的功能
    #********* Begin *********#
    next = conn.zrange('cache:list', 0, 0, withscores=True)
    now = time.time()
    if not next or next[0][1] > now:
        time.sleep(0.1)
    data_id = next[0][0]
    delay = conn.zscore('cache:delay', data_id)
    if delay <= 0:
        conn.zrem('cache:delay', data_id)
        conn.zrem('cache:list', data_id)
        conn.delete('cache:data:' + data_id)
    else:
        data = {'id': data_id, 'data': 'fake data'}
        conn.zadd('cache:list', data_id, now + delay)
        conn.set('cache:data:' + data_id, json.dumps(data))



    #********* End *********#

```

