1. **How to customize each HTTP method (GET/POST/PATCH/DELETE)** in `BookViewSet`
    
2. **How to use `BookViewSet` _without_ a router** (manual `urlpatterns`)
    

---

## 1️⃣ Customizing each request in `BookViewSet`

Your starting point:

`from rest_framework.viewsets import ModelViewSet from .models import Book from .serializers import BookSerializer  class BookViewSet(ModelViewSet):     queryset = Book.objects.all()     serializer_class = BookSerializer`

`ModelViewSet` gives you these actions:

- `list` → GET `/books/`
    
- `retrieve` → GET `/books/<id>/`
    
- `create` → POST `/books/`
    
- `update` → PUT `/books/<id>/`
    
- `partial_update`→ PATCH `/books/<id>/`
    
- `destroy` → DELETE `/books/<id>/`
    

You can override any of them like this:

`from rest_framework.viewsets import ModelViewSet from rest_framework.response import Response from rest_framework import status from .models import Book from .serializers import BookSerializer  class BookViewSet(ModelViewSet):     queryset = Book.objects.all()     serializer_class = BookSerializer      # GET /books/  → list     def list(self, request, *args, **kwargs):         print("Custom list() called")         queryset = self.get_queryset()          # you can filter here if you want:         # queryset = queryset.filter(author__icontains="Martin")          serializer = self.get_serializer(queryset, many=True)         return Response(serializer.data)      # GET /books/<id>/  → retrieve     def retrieve(self, request, *args, **kwargs):         print("Custom retrieve() called")         instance = self.get_object()         serializer = self.get_serializer(instance)         return Response(serializer.data)      # POST /books/  → create     def create(self, request, *args, **kwargs):         print("Custom create() called")         serializer = self.get_serializer(data=request.data)         serializer.is_valid(raise_exception=True)          # you can manipulate validated data here if needed         # validated = serializer.validated_data          self.perform_create(serializer)         return Response(serializer.data, status=status.HTTP_201_CREATED)      # PUT /books/<id>/  → update     def update(self, request, *args, **kwargs):         print("Custom update() called")         partial = kwargs.pop("partial", False)         instance = self.get_object()         serializer = self.get_serializer(instance, data=request.data, partial=partial)         serializer.is_valid(raise_exception=True)         self.perform_update(serializer)         return Response(serializer.data)      # PATCH /books/<id>/  → partial_update     def partial_update(self, request, *args, **kwargs):         print("Custom partial_update() called")         kwargs["partial"] = True         return super().update(request, *args, **kwargs)      # DELETE /books/<id>/  → destroy     def destroy(self, request, *args, **kwargs):         print("Custom destroy() called")         instance = self.get_object()         self.perform_destroy(instance)         return Response(status=status.HTTP_204_NO_CONTENT)`

You don’t _have_ to override all of them—only the ones you want to customize.

---

## 2️⃣ Using `BookViewSet` **without** a router

Right now you have this with a router:

`from django.urls import path, include from rest_framework.routers import DefaultRouter from .views import BookViewSet  router = DefaultRouter() router.register(r'books', BookViewSet, basename='book')  urlpatterns = [     path('api/', include(router.urls)), ]`

You can do the same thing **without** a router in two main ways.

---

### 🔹 Option A: Use `BookViewSet.as_view()` manually

You map HTTP methods to actions yourself.

`books/urls.py`:

`from django.urls import path from .views import BookViewSet  book_list = BookViewSet.as_view({     "get": "list",     "post": "create", })  book_detail = BookViewSet.as_view({     "get": "retrieve",     "put": "update",     "patch": "partial_update",     "delete": "destroy", })  urlpatterns = [     path("api/books/", book_list, name="book-list"),     path("api/books/<int:pk>/", book_detail, name="book-detail"), ]`

This gives you the same behavior as the router, just manually wired:

- `GET /api/books/` → list
    
- `POST /api/books/` → create
    
- `GET /api/books/1/` → retrieve
    
- `PUT /api/books/1/` → update
    
- `PATCH /api/books/1/` → partial_update
    
- `DELETE /api/books/1/` → destroy
    

And your project `urls.py` just includes it like normal:

`from django.urls import path, include  urlpatterns = [     path("admin/", admin.site.urls),     path("", include("books.urls")), ]`

---

### 🔹 Option B: Use class-based generics instead of a ViewSet

If you don’t _need_ a ViewSet, you can split into two DRF generic views:

`books/views.py`:

`from rest_framework import generics from .models import Book from .serializers import BookSerializer  class BookListCreateAPIView(generics.ListCreateAPIView):     queryset = Book.objects.all()     serializer_class = BookSerializer  class BookRetrieveUpdateDestroyAPIView(generics.RetrieveUpdateDestroyAPIView):     queryset = Book.objects.all()     serializer_class = BookSerializer`

`books/urls.py`:

`from django.urls import path from .views import (     BookListCreateAPIView,     BookRetrieveUpdateDestroyAPIView, )  urlpatterns = [     path("api/books/", BookListCreateAPIView.as_view(), name="book-list-create"),     path("api/books/<int:pk>/", BookRetrieveUpdateDestroyAPIView.as_view(), name="book-detail"), ]`

Same endpoints, but no router and no ViewSet. This style feels very explicit and is nice for teaching.

---

## 🔁 Quick recap

### How to customize each method?

Override in `BookViewSet`:

- `list()` → GET collection
    
- `retrieve()` → GET single
    
- `create()` → POST
    
- `update()` → PUT
    
- `partial_update()` → PATCH
    
- `destroy()` → DELETE
    

### How to use without router?

Either:

1. Use `BookViewSet.as_view({ ... })` with manual mappings in `urls.py`, **or**
    
2. Use `ListCreateAPIView` + `RetrieveUpdateDestroyAPIView` and map them in `urls.py`.