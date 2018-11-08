# 明确REST接口开发的核心任务

分析一下上节的案例，可以发现，在开发REST API接口时，视图中做的最主要有三件事：

* 将请求的数据（如JSON格式）转换为模型类对象
* 操作数据库
* 将模型类对象转换为响应的数据（如JSON格式）

## 序列化Serialization

[维基百科](https://zh.wikipedia.org/wiki/%E5%BA%8F%E5%88%97%E5%8C%96)中对于序列化的定义：

**序列化**（serialization）在计算机科学的资料处理中，是指将数据结构或物件状态转换成可取用格式（例如存成档案，存于缓冲，或经由网络中传送），以留待后续在相同或另一台计算机环境中，能恢复原先状态的过程。依照序列化格式重新获取字节的结果时，可以利用它来产生与原始物件相同语义的副本。对于许多物件，像是使用大量参照的复杂物件，这种序列化重建的过程并不容易。面向对象中的物件序列化，并不概括之前原始物件所关联的函式。这种过程也称为物件编组（marshalling）。从一系列字节提取数据结构的反向操作，是反序列化（也称为解编组, deserialization, unmarshalling）。

**序列化**在计算机科学中通常有以下定义:

​	在数据储存与传送的部分是指将一个[对象](https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))存储至一个[储存媒介](https://zh.wikipedia.org/w/index.php?title=%E5%84%B2%E5%AD%98%E5%AA%92%E4%BB%8B&action=edit&redlink=1)，例如[档案](https://zh.wikipedia.org/wiki/%E6%AA%94%E6%A1%88)或是[记亿体缓冲](https://zh.wikipedia.org/w/index.php?title=%E8%A8%98%E5%84%84%E9%AB%94%E7%B7%A9%E8%A1%9D&action=edit&redlink=1)等，或者透过网络传送资料时进行编码的过程，可以是[字节](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82)或是[XML](https://zh.wikipedia.org/wiki/XML)等格式。而[字节](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82)的或[XML](https://zh.wikipedia.org/wiki/XML)编码格式可以还原完全相等的[对象](https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))。这程序被应用在不同[应用程序](https://zh.wikipedia.org/wiki/%E6%87%89%E7%94%A8%E7%A8%8B%E5%BC%8F)之间传送[对象](https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))，以及服务器将[对象](https://zh.wikipedia.org/wiki/%E5%AF%B9%E8%B1%A1_(%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6))储存到[档案](https://zh.wikipedia.org/wiki/%E6%AA%94%E6%A1%88)或[数据库](https://zh.wikipedia.org/wiki/%E8%B3%87%E6%96%99%E5%BA%AB)。相反的过程又称为[**反序列化**](https://zh.wikipedia.org/w/index.php?title=%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96&action=edit&redlink=1)。

简而言之，我们可以将**序列化**理解为：

**将程序中的一个数据结构类型转换为其他格式（字典、JSON、XML等），例如将Django中的模型类对象装换为JSON字符串，这个转换过程我们称为序列化。**

如：

```python
queryset = BookInfo.objects.all()
book_list = []
# 序列化
for book in queryset:
    book_list.append({
        'id': book.id,
        'btitle': book.btitle,
        'bpub_date': book.bpub_date,
        'bread': book.bread,
        'bcomment': book.bcomment,
        'image': book.image.url if book.image else ''
    })
return JsonResponse(book_list, safe=False)
```

**反之，将其他格式（字典、JSON、XML等）转换为程序中的数据，例如将JSON字符串转换为Django中的模型类对象，这个过程我们称为反序列化。**

如：

```python
json_bytes = request.body
json_str = json_bytes.decode()

# 反序列化
book_dict = json.loads(json_str)
book = BookInfo.objects.create(
    btitle=book_dict.get('btitle'),
    bpub_date=datetime.strptime(book_dict.get('bpub_date'), '%Y-%m-%d').date()
)
```

我们可以看到，**在开发REST API时，视图中要频繁的进行序列化与反序列化的编写。**

## 总结

在开发REST API接口时，我们在**视图**中需要做的最核心的事是：

* **将数据库数据序列化为前端所需要的格式，并返回；**

* **将前端发送的数据反序列化为模型类对象，并保存到数据库中。**

  

