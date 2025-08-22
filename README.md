# Apply Pagination with Django Rest Framework(DRF)

## Context:
- [Overview](#overview)
- [Method-1: Global Pagination](#method-1-global-pagination)
- [Method-2: Per-View Pagination](#method-2-per-view-pagination)
- [Method-3: Custom Global Pagination](#method-3-custom-global-pagination)
- [Method-4: Pagination for Function-Based Views](#method-4-pagination-for-function-based-views)
- [Method-5: Pagination for `APIView`:](#method-5-pagination-for-apiview)
- [Method- 1 to 3 Works Automatically For](#method--1-to-3-works-automatically-for)
- [Method- 1 to 3 Does NOT Work Automatically For](#method--1-to-3-does-not-work-automatically-for)
- [How to access Pagination](#how-to-access-pagination)

## Overview
Pagination reduces loading time (both on backend and frontend) and memory usage in most cases, especially when we have a large dataset.  Let break it down:

### ⚡ Without Pagination
- Suppose we have **50,000 materials** in our `MaterialModel`.
- If we call:
    ```
    /api/materials/
    ```
    and pagination is **not enabled**, DRF will:
    
    - Query **all 50,000 rows** from the database.
    - Serialize **all 50,000 objects** into JSON.
    - Send all that data over the network.

👉 This will **slow down the server response**, increase **RAM/CPU usage**, and make the **client app heavy**.

---

### ⚡ With Pagination

Example:

```
/api/materials/?page=1&page_size=20
```

- The query will **only fetch 20 rows** from the database.
- DRF only serializes those 20 objects.
- The response is **much smaller and faster**.

👉 This **reduces server load, speeds up response time, and makes client apps smoother**.


In Django REST Framework (DRF), pagination is not applied inside the ViewSet itself — instead, it’s usually applied globally in our project settings or on a per-view basis.

📒[Go To Context](#context)

## Method-1: Global Pagination
If we want to apply pagination globally for every viewset, then we need to add `pagination_class` inside the `settings.py`

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,  # number of items per page
}
```
Now all our ViewSet will return paginated results automatically, like this:
```json
{
    "count": 25,
    "next": "http://localhost:8000/api/materials/?page=2",
    "previous": null,
    "results": [
        {"id": 1, "name": "Cement", "unit": 1, "unit_name": "Bag", "total_quantity": 500},
        ...
    ]
}
```

> 👉 This applies to all APIs automatically.
>
> `Note:` Here we cann’t define `page_size`. Here `page_size` is always fixed. We only can change manually from `settings.py`. To solve this issue we can apply `Metho-3:` Custom Global Pagination

📒[Go To Context](#context)

## Method-2: Per-View Pagination
If we want pagination only for one endpoint (MaterialViewSet):

```python
from rest_framework.pagination import PageNumberPagination

class MaterialPagination(PageNumberPagination):
    page_size = 10
    page_size_query_param = 'page_size'
    max_page_size = 100

class MaterialViewSet(ModelViewSet):
    queryset = MaterialModel.objects.all()
    serializer_class = MaterialSerializer
    pagination_class = MaterialPagination

```
> 👉 This applies only to MaterialViewSet, not globally.

📒[Go To Context](#context)

## Method-3: Custom Global Pagination
**Step-1:** Create a `pagination.py` file inside the project folder. Then used this code
```python
from rest_framework.pagination import PageNumberPagination

class GlobalPagination(PageNumberPagination):
    page_size = 2
    page_size_query_param = 'page_size'
    max_page_size = 100
```
**Step-2:** Cofigure it inside the `settings.py`:
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'your_directory_name.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
}
```

📒[Go To Context](#context)

## Method-4: Pagination for Function-Based Views
- Global pagination does NOT automatically work for function-based views.
- Reason: DRF applies global pagination only to views that inherit from GenericAPIView (or ViewSets).

```python
from rest_framework.decorators import api_view
from rest_framework.pagination import PageNumberPagination

@api_view(['GET'])
def material_list(request):
    queryset = MaterialModel.objects.all()
    paginator = PageNumberPagination()
    paginator.page_size = 10
    
    paginator.page_size_query_param = 'page_size'
    paginator.max_page_size = 100 
    
    page = paginator.paginate_queryset(queryset, request)

    serializer = MaterialSerializer(page, many=True)
    
    return paginator.get_paginated_response(serializer.data)
```

📒[Go To Context](#context)

## Method-5: Pagination for `APIView`:
```python
class MaterialListAPI(APIView):
    def get(self, request):
        queryset = MaterialModel.objects.all()
        paginator = PageNumberPagination()
        paginator.page_size = 10
        paginator.page_size_query_param = 'page_size'
        paginator.max_page_size = 100
        result_page = paginator.paginate_queryset(queryset, request)
        serializer = MaterialSerializer(result_page, many=True)
        return paginator.get_paginated_response(serializer.data)
```

📒[Go To Context](#context)

## Method- 1 to 3 Works Automatically For

All **DRF views that inherit from `GenericAPIView`**, including:

| View Type | Works Automatically? |
| --- | --- |
| `ModelViewSet` / `ReadOnlyModelViewSet` | ✅ Yes |
| `ListAPIView` | ✅ Yes |
| `ListCreateAPIView` | ✅ Yes |
| Any view inheriting `GenericAPIView` + `ListModelMixin` | ✅ Yes |

📒[Go To Context](#context)

## Method- 1 to 3 Does NOT Work Automatically For

| View Type | Requires Manual Pagination? |
| --- | --- |
| Plain `APIView` | ❌ Yes |
| Function-Based Views (`@api_view`) | ❌ Yes |
| Any custom view that returns a Response directly without using `GenericAPIView` | ❌ Yes |
- For these, we must manually create a paginator, call `paginate_queryset()`, and return `get_paginated_response()`.

📒[Go To Context](#context)

## How to access Pagination

If we want to go to specific page then used this method:

```
http://localhost:8000/api/materials/?page={page_no}

#Example:
http://localhost:8000/api/materials/?page=2
```

Suppose every page if we want to show 20, 30, 100 etc. data. They follow this method:

```
http://localhost:8000/api/materials/?page_size={page_size}

#Example:
http://localhost:8000/api/materials/?page_size=3
```

We can apply `page` and `page_size` at a time

```
http://localhost:8000/api/materials/?page={page_no}&page_size={page_size}

#Example:
http://localhost:8000/api/materials/?page=2&page_size=3
```

📒[Go To Context](#context)
