# 使用Django开发REST 接口

我们以在Django框架中使用的图书英雄案例来写一套支持图书数据增删改查的REST API接口，来理解REST API的开发。

在此案例中，前后端均发送JSON格式数据。

```python
# views.py

from datetime import datetime

class BooksAPIVIew(View):
    """
    查询所有图书、增加图书
    """
    def get(self, request):
        """
        查询所有图书
        路由：GET /books/
        """
        queryset = BookInfo.objects.all()
        book_list = []
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

    def post(self, request):
        """
        新增图书
        路由：POST /books/ 
        """
        json_bytes = request.body
        json_str = json_bytes.decode()
        book_dict = json.loads(json_str)

        # 此处详细的校验参数省略

        book = BookInfo.objects.create(
            btitle=book_dict.get('btitle'),
            bpub_date=datetime.strptime(book_dict.get('bpub_date'), '%Y-%m-%d').date()
        )

        return JsonResponse({
            'id': book.id,
            'btitle': book.btitle,
            'bpub_date': book.bpub_date,
            'bread': book.bread,
            'bcomment': book.bcomment,
            'image': book.image.url if book.image else ''
        }, status=201)


class BookAPIView(View):
    def get(self, request, pk):
        """
        获取单个图书信息
        路由： GET  /books/<pk>/
        """
        try:
            book = BookInfo.objects.get(pk=pk)
        except BookInfo.DoesNotExist:
            return HttpResponse(status=404)

        return JsonResponse({
            'id': book.id,
            'btitle': book.btitle,
            'bpub_date': book.bpub_date,
            'bread': book.bread,
            'bcomment': book.bcomment,
            'image': book.image.url if book.image else ''
        })

    def put(self, request, pk):
        """
        修改图书信息
        路由： PUT  /books/<pk>
        """
        try:
            book = BookInfo.objects.get(pk=pk)
        except BookInfo.DoesNotExist:
            return HttpResponse(status=404)

        json_bytes = request.body
        json_str = json_bytes.decode()
        book_dict = json.loads(json_str)

        # 此处详细的校验参数省略

        book.btitle = book_dict.get('btitle')
        book.bpub_date = datetime.strptime(book_dict.get('bpub_date'), '%Y-%m-%d').date()
        book.save()

        return JsonResponse({
            'id': book.id,
            'btitle': book.btitle,
            'bpub_date': book.bpub_date,
            'bread': book.bread,
            'bcomment': book.bcomment,
            'image': book.image.url if book.image else ''
        })

    def delete(self, request, pk):
        """
        删除图书
        路由： DELETE /books/<pk>/
        """
        try:
            book = BookInfo.objects.get(pk=pk)
        except BookInfo.DoesNotExist:
            return HttpResponse(status=404)

        book.delete()

        return HttpResponse(status=204)
```

```python
# urls.py

urlpatterns = [
    url(r'^books/$', views.BooksAPIVIew.as_view()),
    url(r'^books/(?P<pk>\d+)/$', views.BookAPIView.as_view())
]
```

#### 测试

使用Postman测试上述接口

1） 获取所有图书数据

GET 方式访问 http://127.0.0.1:8000/books/， 返回状态码200，数据如下

```json
[
    {
        "id": 1,
        "btitle": "射雕英雄传",
        "bpub_date": "1980-05-01",
        "bread": 12,
        "bcomment": 34,
        "image": ""
    },
    {
        "id": 2,
        "btitle": "天龙八部",
        "bpub_date": "1986-07-24",
        "bread": 36,
        "bcomment": 40,
        "image": ""
    },
    {
        "id": 3,
        "btitle": "笑傲江湖",
        "bpub_date": "1995-12-24",
        "bread": 20,
        "bcomment": 80,
        "image": ""
    },
    {
        "id": 4,
        "btitle": "雪山飞狐",
        "bpub_date": "1987-11-11",
        "bread": 58,
        "bcomment": 24,
        "image": ""
    },
    {
        "id": 5,
        "btitle": "西游记",
        "bpub_date": "1988-01-01",
        "bread": 10,
        "bcomment": 10,
        "image": "booktest/xiyouji.png"
    },
    {
        "id": 6,
        "btitle": "水浒传",
        "bpub_date": "1992-01-01",
        "bread": 10,
        "bcomment": 11,
        "image": ""
    },
    {
        "id": 7,
        "btitle": "红楼梦",
        "bpub_date": "1990-01-01",
        "bread": 0,
        "bcomment": 0,
        "image": ""
    }
]
```

2）获取单一图书数据

GET  访问 http://127.0.0.1:8000/books/5/ ，返回状态码200， 数据如下

```json
{
    "id": 5,
    "btitle": "西游记",
    "bpub_date": "1988-01-01",
    "bread": 10,
    "bcomment": 10,
    "image": "booktest/xiyouji.png"
}
```

GET  访问http://127.0.0.1:8000/books/100/，返回状态码404

3）新增图书数据

POST 访问http://127.0.0.1:8000/books/，发送JSON数据：

```json
{
	"btitle": "三国演义",
	"bpub_date": "1990-02-03"
}
```

返回状态码201，数据如下

```json
{
    "id": 8,
    "btitle": "三国演义",
    "bpub_date": "1990-02-03",
    "bread": 0,
    "bcomment": 0,
    "image": ""
}
```

4）修改图书数据

PUT 访问http://127.0.0.1:8000/books/8/，发送JSON数据：

```json
{
	"btitle": "三国演义（第二版）",
	"bpub_date": "1990-02-03"
}
```

返回状态码200，数据如下

```json
{
    "id": 8,
    "btitle": "三国演义（第二版）",
    "bpub_date": "1990-02-03",
    "bread": 0,
    "bcomment": 0,
    "image": ""
}
```

5）删除图书数据

DELETE 访问http://127.0.0.1:8000/books/8/，返回204状态码