
# Django REST Framework (DRF) এর ViewSet: একটি বিস্তারিত ব্যাখ্যা

সাধারণত, Django REST Framework-এ আমরা প্রতিটি কাজের জন্য আলাদা আলাদা ভিউ (View) তৈরি করি। যেমন: একটি আইটেমের তালিকা দেখানোর জন্য `ItemListAPIView`, একটি নির্দিষ্ট আইটেম দেখানোর জন্য `ItemDetailAPIView` ইত্যাদি। এতে কোড পুনরাবৃত্তি (code repetition) হয় এবং কোড ম্যানেজ করা কঠিন হয়ে পড়ে।

এই সমস্যার সমাধান হলো **ViewSet**।

## ViewSet কী?

`ViewSet` হলো একটি বিশেষ ধরণের ক্লাস যা একাধিক সম্পর্কিত ভিউ-এর লজিককে একটিমাত্র ক্লাসে একত্রিত করে। এটি আপনাকে `list`, `create`, `retrieve`, `update`, `partial_update`, এবং `destroy` এর মতো সাধারণ CRUD (Create, Read, Update, Delete) অপারেশনগুলো একটি ক্লাসের মধ্যেই লেখার সুযোগ দেয়।

সহজ কথায়, আপনাকে আর `ListAPIView`, `CreateAPIView`, `RetrieveUpdateDestroyAPIView`-এর মতো একাধিক ক্লাস তৈরি করতে হবে না। একটি `ModelViewSet` ব্যবহার করেই আপনি এই সব কাজ করতে পারবেন। এর প্রধান সুবিধা হলো **DRY (Don't Repeat Yourself)** নীতি অনুসরণ করা, অর্থাৎ কোড কম লেখা এবং গোছানো রাখা।


## প্রধান বৈশিষ্ট্যসমূহ (Key Features)

`ViewSet`-এর কয়েকটি মূল ধারণা নিচে তুলে ধরা হলো:

### ১. অ্যাকশন (Actions)

`ViewSet`-এ সাধারণ `get()`, `post()`, `put()` মেথডের পরিবর্তে **অ্যাকশন (action)** ব্যবহার করা হয়। এই অ্যাকশনগুলো হলো:

-   `.list()`: একাধিক অবজেক্টের একটি তালিকা দেখায় (HTTP `GET`)।
-   `.retrieve()`: একটি নির্দিষ্ট অবজেক্ট দেখায় (HTTP `GET`)।
-   `.create()`: একটি নতুন অবজেক্ট তৈরি করে (HTTP `POST`)।
-   `.update()`: একটি অবজেক্টকে পুরোপুরি আপডেট করে (HTTP `PUT`)।
-   `.partial_update()`: একটি অবজেক্টকে আংশিকভাবে আপডেট করে (HTTP `PATCH`)।
-   `.destroy()`: একটি অবজেক্ট মুছে ফেলে (HTTP `DELETE`)।

**উদাহরণ:**

```python
# views.py
from rest_framework import viewsets
from rest_framework.response import Response
from django.shortcuts import get_object_or_404
from myapp.models import User
from myapp.serializers import UserSerializer

class UserViewSet(viewsets.ViewSet):
    """
    একটি সাধারণ ViewSet যা ব্যবহারকারীদের তালিকা দেখায় বা নির্দিষ্ট ব্যবহারকারীকে দেখায়।
    """
    def list(self, request):
        queryset = User.objects.all()
        serializer = UserSerializer(queryset, many=True)
        return Response(serializer.data)

    def retrieve(self, request, pk=None):
        queryset = User.objects.all()
        user = get_object_or_404(queryset, pk=pk)
        serializer = UserSerializer(user)
        return Response(serializer.data)
```

### ২. URL এর সাথে ViewSet যুক্ত করা (Binding ViewSets to URLs)

`ViewSet` নিজে থেকে URL হ্যান্ডেল করে না। একে URL-এর সাথে যুক্ত করতে হয়। এর দুটি উপায় আছে:

#### ক) ম্যানুয়াল পদ্ধতি (Explicitly binding)

আপনি `as_view()` মেথড ব্যবহার করে কোন HTTP মেথড কোন অ্যাকশনের সাথে যুক্ত হবে তা বলে দিতে পারেন।

**urls.py:**
```python
# urls.py
from myapp.views import UserViewSet
from django.urls import path

# UserViewSet থেকে দুটি আলাদা ভিউ তৈরি করা হচ্ছে
user_list = UserViewSet.as_view({'get': 'list'})
user_detail = UserViewSet.as_view({'get': 'retrieve'})

urlpatterns = [
    path('users/', user_list, name='user-list'),
    path('users/<int:pk>/', user_detail, name='user-detail'),
]
```
এখানে, `GET` রিকোয়েস্ট `/users/` URL-এ আসলে `list` অ্যাকশনটি কল হবে এবং `/users/<pk>/` URL-এ আসলে `retrieve` অ্যাকশনটি কল হবে।

#### খ) স্বয়ংক্রিয় পদ্ধতি (Using Routers)

এটিই `ViewSet` ব্যবহারের সবচেয়ে সহজ এবং জনপ্রিয় উপায়। DRF-এর `Router` ক্লাস ব্যবহার করে আপনি স্বয়ংক্রিয়ভাবে `ViewSet`-এর জন্য URL তৈরি করতে পারেন।

**urls.py:**
```python
# urls.py
from django.urls import path, include
from rest_framework.routers import SimpleRouter
from myapp.views import UserViewSet

# একটি রাউটার তৈরি করা হচ্ছে
router = SimpleRouter()
# UserViewSet-কে 'users' নামে রেজিস্টার করা হচ্ছে
router.register(r'users', UserViewSet, basename='user')

# রাউটার দ্বারা তৈরি করা URL-গুলো urlpatterns-এ যুক্ত করা হচ্ছে
urlpatterns = [
    path('', include(router.urls)),
]
```

এই সামান্য কোডটুকু লিখলেই `SimpleRouter` স্বয়ংক্রিয়ভাবে নিচের URL-গুলো তৈরি করে দেবে:
-   `GET /users/` -> `list()`
-   `POST /users/` -> `create()`
-   `GET /users/{pk}/` -> `retrieve()`
-   `PUT /users/{pk}/` -> `update()`
-   `PATCH /users/{pk}/` -> `partial_update()`
-   `DELETE /users/{pk}/` -> `destroy()`

`DefaultRouter` নামে আরেকটি রাউটার আছে, যা `SimpleRouter`-এর মতোই কাজ করে এবং সাথে একটি ডিফল্ট API রুট ভিউ (`/`) তৈরি করে দেয়।

### ৩. অতিরিক্ত অ্যাকশন তৈরি করা (`@action` decorator)

CRUD অপারেশনের বাইরেও আপনার কাস্টম অ্যাকশনের প্রয়োজন হতে পারে। যেমন: একজন ইউজারের জন্য পাসওয়ার্ড সেট করা। এর জন্য `@action` ডেকোরেটর ব্যবহার করা হয়।

**উদাহরণ:** একজন ইউজারের পাসওয়ার্ড পরিবর্তন করার জন্য একটি কাস্টম অ্যাকশন।

```python
# views.py
from rest_framework.decorators import action
from rest_framework.response import Response

class UserViewSet(viewsets.ModelViewSet):
    # ... queryset এবং serializer_class ...

    @action(detail=True, methods=['post'], url_path='set-password')
    def set_password(self, request, pk=None):
        user = self.get_object()
        # ... পাসওয়ার্ড সেট করার লজিক এখানে থাকবে ...
        return Response({'status': 'password set'})
```

-   `@action(detail=True, ...)`: এটি একটি নির্দিষ্ট অবজেক্টের উপর কাজ করবে। এর URL হবে `/users/{pk}/set-password/`।
-   `@action(detail=False, ...)`: এটি পুরো তালিকার উপর কাজ করবে (যেমন `/users/active-users/`)।
-   `methods=['post']`: এটি নির্দিষ্ট করে যে এই অ্যাকশনটি শুধু `POST` রিকোয়েস্ট গ্রহণ করবে।

---

## বিভিন্ন ধরণের ViewSet

DRF-এ কয়েকটি বিল্ট-ইন `ViewSet` ক্লাস রয়েছে:

1.  **`ViewSet`**:
    -   সবচেয়ে বেসিক ক্লাস। এটি `APIView` থেকে ইনহেরিট করে।
    -   এতে ডিফল্টভাবে কোনো অ্যাকশন (`list`, `create` ইত্যাদি) থাকে না। আপনাকে নিজে সেগুলো ইমপ্লিমেন্ট করতে হবে।

2.  **`GenericViewSet`**:
    -   এটি `GenericAPIView` থেকে ইনহেরিট করে।
    -   এটি `get_object()` এবং `get_queryset()` এর মতো বেসিক মেথডগুলো দেয়, কিন্তু কোনো অ্যাকশন ডিফল্টভাবে ইমপ্লিমেন্ট করা থাকে না।
    -   আপনি মিক্সিন (Mixin) ক্লাস (`CreateModelMixin`, `ListModelMixin` ইত্যাদি) যোগ করে প্রয়োজন অনুযায়ী অ্যাকশন যুক্ত করতে পারেন।

3.  **`ReadOnlyModelViewSet`**:
    -   এটি শুধুমাত্র "পড়ার" (read-only) কাজগুলো করে।
    -   এতে ডিফল্টভাবে `.list()` এবং `.retrieve()` অ্যাকশন দুটি থাকে।
    -   যেই API দিয়ে শুধু ডেটা দেখানো হবে, সেখানে এটি খুব কার্যকরী।

4.  **`ModelViewSet`**:
    -   এটি সবচেয়ে শক্তিশালী এবং সর্বাধিক ব্যবহৃত `ViewSet`।
    -   এতে সম্পূর্ণ CRUD অপারেশনের জন্য সব অ্যাকশন (`list`, `create`, `retrieve`, `update`, `partial_update`, `destroy`) ইমপ্লিমেন্ট করা থাকে।
    -   একটি Django মডেলের জন্য একটি সম্পূর্ণ API তৈরি করতে এটি ব্যবহার করা হয়। আপনাকে শুধু `queryset` এবং `serializer_class` বলে দিতে হয়।

**`ModelViewSet`-এর উদাহরণ:**

```python
# views.py
from rest_framework import viewsets
from myapp.models import User
from myapp.serializers import UserSerializer

class UserViewSet(viewsets.ModelViewSet):
    """
    এই ViewSet স্বয়ংক্রিয়ভাবে 'list', 'create', 'retrieve',
    'update' এবং 'destroy' অ্যাকশন প্রদান করে।
    """
    queryset = User.objects.all()
    serializer_class = UserSerializer
```
এই সামান্য কোডটুকুই একটি সম্পূর্ণ User API তৈরি করার জন্য যথেষ্ট!

---

## সারসংক্ষেপ

*   **ViewSet** কোডকে গোছানো রাখে এবং পুনরাবৃত্তি কমায়।
*   এটি **অ্যাকশন** (`list`, `retrieve` ইত্যাদি) ব্যবহার করে একাধিক ভিউ-এর কাজ একটি ক্লাসে করে।
*   **Router** ব্যবহার করে `ViewSet`-এর জন্য URL তৈরি করা খুবই সহজ এবং কার্যকর।
*   সাধারণ CRUD অপারেশনের জন্য **`ModelViewSet`** ব্যবহার করা সবচেয়ে ভালো।
*   কাস্টম কাজের জন্য `@action` ডেকোরেটর ব্যবহার করা যায়।

> এই ব্যাখ্যাটি [Django REST Framework-এর অফিশিয়াল ডকুমেন্টেশন](https://www.django-rest-framework.org/api-guide/viewsets/) অবলম্বনে তৈরি করা হয়েছে।