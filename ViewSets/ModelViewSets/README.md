# Django REST Framework-এ `ModelViewSet`

কল্পনা করুন, আপনাকে একটি সম্পূর্ণ API তৈরি করতে হবে যা দিয়ে ব্যবহারকারীরা বইয়ের তথ্য দেখতে, নতুন বই যোগ করতে, আপডেট করতে এবং মুছে ফেলতে পারবে। এই সবগুলো কাজের জন্য আলাদা আলাদা ভিউ লেখা বেশ সময়সাপেক্ষ।

 এখানেই Django REST Framework (DRF)-এর ম্যাজিক শুরু হয়, আর সেই ম্যাজিকের নাম হলো **`ModelViewSet`**।

## `ModelViewSet` কী?

সহজ ভাষায়, `ModelViewSet` হলো DRF-এর একটি "সুইস আর্মি নাইফ"। এটি এমন একটি ক্লাস যা একটি Django মডেলের জন্য সম্পূর্ণ CRUD (Create, Read, Update, Delete) কার্যকারিতা স্বয়ংক্রিয়ভাবে তৈরি করে দেয়। আপনাকে শুধু বলে দিতে হবে কোন মডেল এবং কোন সিরিয়ালাইজার ব্যবহার করতে হবে, বাকি কাজ `ModelViewSet` নিজেই করে নেবে।

এটি ডিফল্টভাবে নিচের অ্যাকশনগুলো প্রদান করে:
-   `.list()`: সব অবজেক্টের তালিকা দেখানো (GET)।
-   `.create()`: নতুন অবজেক্ট তৈরি করা (POST)।
-   `.retrieve()`: একটি নির্দিষ্ট অবজেক্ট দেখানো (GET by ID)।
-   `.update()`: একটি অবজেক্টকে সম্পূর্ণ আপডেট করা (PUT)।
-   `.partial_update()`: একটি অবজেক্টকে আংশিকভাবে আপডেট করা (PATCH)।
-   `.destroy()`: একটি অবজেক্ট মুছে ফেলা (DELETE)।


## `ModelViewSet` কেন অন্যান্য ভিউ থেকে আলাদা?

`ModelViewSet`-কে ভালোভাবে বুঝতে হলে একে `APIView`, `ViewSet`, এবং `GenericViewSet`-এর সাথে তুলনা করতে হবে।

| বৈশিষ্ট্য (Feature)         | `APIView`                               | `ViewSet`                                     | `GenericViewSet`                                    | `ModelViewSet`                                                     |
| -------------------------- | --------------------------------------- | --------------------------------------------- | --------------------------------------------------- | ------------------------------------------------------------------ |
| **মূল ভিত্তি**             | Django-র `View` ক্লাস                   | `APIView`                                     | `GenericAPIView`                                    | `GenericViewSet` ও সকল Mixins                                     |
| **HTTP মেথড হ্যান্ডলিং**     | নিজেকে লিখতে হয় (`get`, `post` ইত্যাদি) | অ্যাকশন (`list`, `create`) ও HTTP মেথড বাইন্ড করতে হয় | অ্যাকশন বাইন্ড করতে হয়                               | **স্বয়ংক্রিয়ভাবে** সব CRUD অ্যাকশন যুক্ত থাকে                     |
| **বেসিক লজিক**              | নিজেকে সব লজিক লিখতে হয়                | নিজেকে সব লজিক লিখতে হয়                        | অবজেক্ট পাওয়ার (`get_object`) বেসিক লজিক দেয়        | **সম্পূর্ণ CRUD লজিক** দেওয়া থাকে                                  |
| **কোডের পরিমাণ**            | অনেক বেশি                             | মাঝারি                                         | কম                                                  | **সবচেয়ে কম**                                                      |
| **ব্যবহারের ক্ষেত্র**        | খুব কাস্টম বা জটিল লজিকের জন্য           | রাউটারের সুবিধা নিয়ে কাস্টম লজিক লেখার জন্য     | Mixin ব্যবহার করে নির্দিষ্ট কিছু অ্যাকশন বানানোর জন্য | **দ্রুত CRUD API** তৈরি করার জন্য (সবচেয়ে সাধারণ ব্যবহার)         |

**মূল কথা:** আপনি `APIView` থেকে `ModelViewSet`-এর দিকে যতই যাবেন, DRF আপনার জন্য তত বেশি কাজ স্বয়ংক্রিয়ভাবে করে দেবে, এবং আপনার কোড লেখার পরিমাণও তত কমে যাবে।

---

## `ModelViewSet`-এর গুরুত্বপূর্ণ অংশগুলো

একটি `ModelViewSet` ব্যবহার করার সময় কয়েকটি গুরুত্বপূর্ণ প্রপার্টি ও মেথড সম্পর্কে জানা আবশ্যক।

#### ۱. `queryset`
এটি `ModelViewSet`-কে বলে দেয় যে কোন ডেটাসেটের উপর কাজ করতে হবে। এটি সাধারণত `Model.objects.all()` হয়।

**উদাহরণ:**
```python
queryset = Book.objects.all()
```

#### ۲. `serializer_class`
এটি `ModelViewSet`-কে বলে দেয় যে ডেটাগুলোকে JSON বা অন্য ফরম্যাটে রূপান্তর এবং ভ্যালিডেট করার জন্য কোন সিরিয়ালাইজার ব্যবহার করতে হবে।

**উদাহরণ:**
```python
serializer_class = BookSerializer
```

#### ۳. `get_queryset()`
কখনও কখনও আপনাকে ডাইনামিকভাবে `queryset` ফিল্টার করতে হতে পারে। যেমন, শুধুমাত্র লগইন করা ব্যবহারকারীর তৈরি করা বইগুলো দেখানো। এক্ষেত্রে `queryset` প্রপার্টি ব্যবহার না করে `get_queryset()` মেথড ওভাররাইড করতে হয়।

**উদাহরণ:** শুধুমাত্র অ্যাক্টিভ বই দেখানোর জন্য।
```python
def get_queryset(self):
    """
    এই ভিউ শুধুমাত্র is_active=True থাকা বইগুলো রিটার্ন করবে।
    """
    return Book.objects.filter(is_active=True)
```

#### ۴. `perform_create()`
যখন একটি নতুন অবজেক্ট তৈরি করা হয় (POST রিকোয়েস্টের মাধ্যমে), তখন ডেটা সেভ করার ঠিক আগে আপনি যদি অতিরিক্ত কোনো কাজ করতে চান, তাহলে `perform_create()` মেথড ব্যবহার করতে পারেন। যেমন, স্বয়ংক্রিয়ভাবে লেখকের নাম হিসেবে লগইন করা ব্যবহারকারীকে সেট করা।

**উদাহরণ:**
```python
def perform_create(self, serializer):
    """
    নতুন বই তৈরি করার সময় লেখক হিসেবে বর্তমান ব্যবহারকারীকে সেট করা হবে।
    """
    # owner ফিল্ডে বর্তমান ইউজারকে সেট করে অবজেক্ট সেভ করা হচ্ছে
    serializer.save(owner=self.request.user)
```

---

## একটি সম্পূর্ণ উদাহরণ (বইয়ের API)

আসুন, একটি বইয়ের তথ্য ম্যানেজ করার জন্য একটি সম্পূর্ণ API তৈরি করি।

#### ধাপ ১: মডেল তৈরি (`models.py`)

প্রথমে `books` নামে একটি অ্যাপ তৈরি করে তার `models.py` ফাইলে নিচের মডেলটি লিখুন।

```python
# books/models.py
from django.db import models
from django.contrib.auth.models import User

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    publication_date = models.DateField()
    owner = models.ForeignKey(User, related_name='books', on_delete=models.CASCADE, null=True) # বইটির মালিক

    def __str__(self):
        return self.title
```

#### ধাপ ২: সিরিয়ালাইজার তৈরি (`serializers.py`)

এখন `books` অ্যাপের ভেতরে `serializers.py` নামে একটি ফাইল তৈরি করে নিচের কোডটি লিখুন।

```python
# books/serializers.py
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = ['id', 'title', 'author', 'publication_date', 'owner']
        read_only_fields = ['owner'] # owner ফিল্ডটি শুধুমাত্র পড়ার জন্য
```

> **নোট:** `read_only_fields = ['owner']` দিয়ে আমরা নিশ্চিত করছি যে ব্যবহারকারী API-এর মাধ্যমে `owner` ফিল্ড পরিবর্তন করতে পারবে না। এটি `perform_create`-এর মাধ্যমে স্বয়ংক্রিয়ভাবে সেট হবে।

#### ধাপ ৩: `ModelViewSet` তৈরি (`views.py`)

এখন `books/views.py` ফাইলে আমাদের `ModelViewSet` তৈরি করার পালা।

```python
# books/views.py
from rest_framework import viewsets, permissions
from .models import Book
from .serializers import BookSerializer

class BookViewSet(viewsets.ModelViewSet):
    """
    এই ViewSet টি বইয়ের জন্য 'list', 'create', 'retrieve',
    'update' এবং 'destroy' অ্যাকশন স্বয়ংক্রিয়ভাবে প্রদান করে।
    """
    queryset = Book.objects.all()
    serializer_class = BookSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly] # শুধুমাত্র লগইন করা ব্যবহারকারী লিখতে পারবে

    def perform_create(self, serializer):
        """নতুন বই তৈরি করার সময় লেখক হিসেবে বর্তমান ব্যবহারকারীকে সেট করা হবে।"""
        serializer.save(owner=self.request.user)
```

দেখুন, কত অল্প কোড! এই একটি ক্লাসই আমাদের সম্পূর্ণ CRUD API-এর কাজ করে দিচ্ছে।

#### ধাপ ৪: URL কনফিগারেশন (`urls.py`)

সবশেষে, প্রজেক্টের প্রধান `urls.py` ফাইলে DRF-এর `Router` ব্যবহার করে এই `ViewSet`-এর জন্য URL তৈরি করতে হবে।

```python
# myproject/urls.py
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from books.views import BookViewSet

# একটি রাউটার তৈরি করা হচ্ছে
router = DefaultRouter()
# BookViewSet কে 'books' নামে রেজিস্টার করা হচ্ছে
router.register(r'books', BookViewSet, basename='book')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)), # আমাদের API URL গুলো এখানে যুক্ত হলো
]
```

ব্যাস, কাজ শেষ! এখন `Router` স্বয়ংক্রিয়ভাবে নিচের URL গুলো তৈরি করে দেবে:
-   `GET /api/books/`: সব বইয়ের তালিকা।
-   `POST /api/books/`: নতুন বই তৈরি করা।
-   `GET /api/books/{id}/`: নির্দিষ্ট একটি বই দেখা।
-   `PUT /api/books/{id}/`: নির্দিষ্ট বই আপডেট করা।
-   `PATCH /api/books/{id}/`: নির্দিষ্ট বই আংশিকভাবে আপডেট করা।
-   `DELETE /api/books/{id}/`: নির্দিষ্ট বই মুছে ফেলা।

## সারসংক্ষেপ

`ModelViewSet` হলো Django REST Framework-এর অন্যতম শক্তিশালী একটি টুল যা আপনাকে দ্রুত এবং কার্যকরভাবে স্ট্যান্ডার্ড CRUD API তৈরি করতে সাহায্য করে। কম কোড লিখে প্রচলিত নিয়ম (convention) অনুসরণ করে API তৈরির জন্য এটি সেরা উপায়। 

এই গাইডটি যদি আপনার উপকারে আসে, তাহলে উপরে একটি ⭐️ স্টার দিতে ভুলবেন না!  
আরো ভালো করতে বা নতুন কিছু যোগ করতে চাইলে অবদান (contribute) করতে পারেন।  
আপনার লাইক ও অবদান আমাদের আরও ভালো কনটেন্ট তৈরি করতে উৎসাহিত করবে! 😊