
# Django REST Framework (DRF) এ ফিল্টারিং (Filtering)

যখন আমরা একটি API তৈরি করি, তখন ক্লায়েন্ট বা ব্যবহারকারীকে প্রায়শই সম্পূর্ণ ডেটাসেটের একটি নির্দিষ্ট অংশ দেখার প্রয়োজন হয়। যেমন, একজন ব্যবহারকারী হয়তো শুধু নির্দিষ্ট ক্যাটাগরির পণ্য দেখতে চায়, অথবা একটি নির্দিষ্ট দামের মধ্যে থাকা পণ্যগুলো খুঁজতে চায়। এই কাজটি করার পদ্ধতিকেই **ফিল্টারিং** বলা হয়।

DRF আপনাকে খুব সহজেই এই ফিল্টারিং ব্যবস্থা যুক্ত করার সুযোগ দেয়।

## ফিল্টারিং কেন প্রয়োজন?

-   **পারফরম্যান্স:** ক্লায়েন্টকে অপ্রয়োজনীয় ডেটা পাঠানো থেকে বিরত রাখে, ফলে API দ্রুত রেসপন্স করে।
-   **ব্যবহারযোগ্যতা:** ব্যবহারকারীরা তাদের প্রয়োজন অনুযায়ী সুনির্দিষ্ট ডেটা খুঁজে পেতে পারে।
-   **নমনীয়তা (Flexibility):** ফ্রন্টএন্ড ডেভেলপারদের জন্য API ব্যবহার করা অনেক সহজ হয়ে যায়।

## DRF-এ ফিল্টারিং কীভাবে কাজ করে?

DRF-এ ফিল্টারিং যুক্ত করার মূল ভিত্তি হলো `filter_backends`। এটি আপনার `View` বা `ViewSet`-এর একটি অ্যাট্রিবিউট, যেখানে আপনি কোন কোন ফিল্টারিং ক্লাস ব্যবহার করতে চান তা বলে দেন।

```python
# views.py
from rest_framework import generics
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [SomeFilterBackend, AnotherFilterBackend] # এখানে ফিল্টার ক্লাসগুলো যুক্ত করতে হয়
```

DRF ডিফল্টভাবে কয়েকটি শক্তিশালী ফিল্টারিং ব্যাকএন্ড প্রদান করে। চলুন, সবচেয়ে গুরুত্বপূর্ণগুলো নিয়ে আলোচনা করা যাক।

---

### ১. `DjangoFilterBackend` (সবচেয়ে শক্তিশালী এবং জনপ্রিয়)

এটি সবচেয়ে নমনীয় এবং বহুল ব্যবহৃত ফিল্টারিং পদ্ধতি। এটি ব্যবহারের জন্য আপনাকে `django-filter` নামে একটি প্যাকেজ ইনস্টল করতে হবে।

**ধাপ ১: ইনস্টলেশন**
```bash
pip install django-filter
```

**ধাপ ২: `settings.py`-তে যুক্ত করা**
আপনার প্রজেক্টের `settings.py` ফাইলে `INSTALLED_APPS`-এর মধ্যে `'django_filters'` যোগ করুন।

```python
# settings.py
INSTALLED_APPS = [
    ...
    'rest_framework',
    'django_filters', # এখানে যোগ করুন
    ...
]
```

এখন আপনি আপনার ভিউতে `DjangoFilterBackend` ব্যবহার করতে পারবেন।

#### ক) সহজ ব্যবহার: `filterset_fields`

এটি সবচেয়ে সহজ উপায়। আপনাকে শুধু বলে দিতে হবে কোন কোন ফিল্ডের উপর ভিত্তি করে ফিল্টার করতে চান।

**উদাহরণ:** একটি পণ্যের `category` এবং `in_stock` স্ট্যাটাস দিয়ে ফিল্টার করা।

```python
# views.py
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import generics
from .models import Product
from .serializers import ProductSerializer

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['category', 'in_stock'] # কোন কোন ফিল্ড দিয়ে ফিল্টার হবে
```

এখন আপনি URL-এর মাধ্যমে এভাবে ফিল্টার করতে পারবেন:
-   `http://127.0.0.1:8000/api/products/?category=electronics`
-   `http://127.0.0.1:8000/api/products/?in_stock=true`
-   `http://127.0.0.1:8000/api/products/?category=books&in_stock=true`

#### খ) উন্নত ব্যবহার: `FilterSet` ক্লাস

আরও জটিল এবং কাস্টম ফিল্টারিং-এর জন্য `FilterSet` ক্লাস ব্যবহার করা হয়। যেমন: নামের মধ্যে আংশিক মিল খোঁজা, দামের একটি নির্দিষ্ট রেঞ্জের মধ্যে পণ্য খোঁজা ইত্যাদি।

**ধাপ ১: `filters.py` ফাইল তৈরি**
আপনার অ্যাপের ভেতরে `filters.py` নামে একটি ফাইল তৈরি করুন।

```python
# products/filters.py
from django_filters import rest_framework as filters
from .models import Product

class ProductFilter(filters.FilterSet):
    # দামের জন্য min এবং max রেঞ্জ ফিল্টার
    min_price = filters.NumberFilter(field_name="price", lookup_expr='gte') # gte = greater than or equal
    max_price = filters.NumberFilter(field_name="price", lookup_expr='lte') # lte = less than or equal

    # নামের মধ্যে insensitive search (ছোট/বড় হাতের অক্ষর気に করবে না)
    name = filters.CharFilter(field_name="name", lookup_expr='icontains')

    class Meta:
        model = Product
        fields = ['category', 'name', 'min_price', 'max_price']
```

**ধাপ ২: ভিউতে `FilterSet` ক্লাস ব্যবহার করা**

```python
# views.py
from .filters import ProductFilter # আমাদের বানানো ফিল্টার ক্লাস ইম্পোর্ট করি

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_class = ProductFilter # filterset_fields এর পরিবর্তে filterset_class ব্যবহার করি
```

এখন আপনি আরও শক্তিশালী কোয়েরি চালাতে পারবেন:
-   `?min_price=500&max_price=2000` (৫০০ থেকে ২০০০ টাকার মধ্যে পণ্য)
-   `?name=laptop` (যেসব পণ্যের নামে "laptop" শব্দটি আছে)
-   `?category=electronics&min_price=50000`

---

### ২. `SearchFilter` (সার্চ করার জন্য)

এই ফিল্টারটি ব্যবহারকারীদের একটি সাধারণ সার্চ বক্সের মতো সুবিধা দেয়। এটি নির্দিষ্ট কিছু ফিল্ডে টেক্সট সার্চ করার জন্য ব্যবহৃত হয়।

**উদাহরণ:** পণ্যের `name` এবং `description` ফিল্ডে সার্চ করা।

```python
# views.py
from rest_framework import filters

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [filters.SearchFilter]
    search_fields = ['name', 'description'] # কোন কোন ফিল্ডে সার্চ হবে
```

এখন ব্যবহারকারী URL-এ `search` প্যারামিটার ব্যবহার করে সার্চ করতে পারবে:
-   `http://127.0.0.1:8000/api/products/?search=gaming laptop`

> **নোট:** আপনি `search_fields`-এ বিভিন্ন lookup prefix ব্যবহার করতে পারেন, যেমন: `'^name'` (নামের শুরু দিয়ে সার্চ), `'=name'` (সম্পূর্ণ মিল), ইত্যাদি।

---

### ৩. `OrderingFilter` (সাজানোর জন্য)

এই ফিল্টারটি ব্যবহারকারীদের ফলাফলের তালিকা সাজানোর (ordering/sorting) সুযোগ দেয়। যেমন, দাম অনুযায়ী ছোট থেকে বড় বা বড় থেকে ছোট করে সাজানো।

**উদাহরণ:** পণ্যের `name` এবং `price` অনুযায়ী সাজানো।

```python
# views.py
from rest_framework import filters

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [filters.OrderingFilter]
    ordering_fields = ['name', 'price'] # কোন কোন ফিল্ড দিয়ে সাজানো যাবে
```

এখন ব্যবহারকারী `ordering` প্যারামিটার ব্যবহার করে ফলাফল সাজাতে পারবে:
-   `?ordering=price` (দাম অনুযায়ী ছোট থেকে বড়)
-   `?ordering=-price` (দাম অনুযায়ী বড় থেকে ছোট)
-   `?ordering=name`

আপনি চাইলে একটি ডিফল্ট অর্ডারিংও সেট করতে পারেন:
```python
class ProductListView(generics.ListAPIView):
    # ...
    ordering = ['-price'] # ডিফল্টভাবে দামের বড় থেকে ছোট ক্রমে দেখাবে
```

---

### সবগুলোকে একসাথে ব্যবহার

আপনি চাইলে এই সবগুলো ফিল্টার একসাথে একটি ভিউতে ব্যবহার করতে পারেন।

```python
# views.py
from django_filters.rest_framework import DjangoFilterBackend
from rest_framework import filters, generics

class ProductListView(generics.ListAPIView):
    queryset = Product.objects.all()
    serializer_class = ProductSerializer
    filter_backends = [DjangoFilterBackend, filters.SearchFilter, filters.OrderingFilter]
    
    # DjangoFilterBackend এর জন্য
    filterset_fields = ['category', 'in_stock']
    
    # SearchFilter এর জন্য
    search_fields = ['name', 'description']
    
    # OrderingFilter এর জন্য
    ordering_fields = ['price', 'name']
```

এখন আপনি একটি মাত্র URL-এই একাধিক কাজ করতে পারবেন:
`?category=electronics&search=dell&ordering=-price`

এই URL-টি "electronics" ক্যাটাগরির মধ্যে "dell" শব্দটি আছে এমন পণ্যগুলোকে খুঁজবে এবং সেগুলোকে দামের বড় থেকে ছোট ক্রমে সাজিয়ে দেখাবে।

## সারসংক্ষেপ

| ফিল্টার ক্লাস         | কাজ                                                     | কখন ব্যবহার করবেন?                                                                     |
| -------------------- | -------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `DjangoFilterBackend` | নির্দিষ্ট ফিল্ডের মানের উপর ভিত্তি করে ফিল্টার করা         | যখন সুনির্দিষ্ট ফিল্ডের (যেমন category, status) উপর ভিত্তি করে ডেটা খুঁজতে হবে।        |
| `SearchFilter`       | এক বা একাধিক ফিল্ডে টেক্সট সার্চ করা                     | যখন একটি সাধারণ সার্চ বক্সের মতো কার্যকারিতা প্রয়োজন।                                    |
| `OrderingFilter`     | ফলাফলের তালিকা সাজানো (Ascending/Descending)             | যখন ব্যবহারকারীকে ডেটা সর্ট করার (যেমন নাম, দাম, তারিখ অনুযায়ী) সুবিধা দিতে হবে। |

ফিল্টারিং DRF-এর একটি অপরিহার্য অংশ যা আপনার API-কে অনেক বেশি কার্যকরী এবং ব্যবহারকারী-বান্ধব করে তোলে।


---

## ভালো লাগলে...

-   **Star দিন:** এই প্রজেক্টটি ভালো লাগলে গিটহাবে Star দিন ⭐
-   **শেয়ার করুন:** বন্ধুদের সাথে শেয়ার করুন, যাতে তারাও শিখতে পারে।
-   **Contribute করুন:** কোনো ভুল, নতুন আইডিয়া থাকলে Pull Request পাঠান!

শুভকামনা! 😊
