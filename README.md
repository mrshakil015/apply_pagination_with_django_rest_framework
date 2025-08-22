# apply_pagination_with_django_rest_framework
# Apply Pagination with Django Rest Framework(DRF)

In Django REST Framework (DRF), pagination is not applied inside the ViewSet itself — instead, it’s usually applied globally in our project settings or on a per-view basis.

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
> `Note:` Here you cann’t define `page_size`. Here `page_size` is always fixed. You only can change manually from `settings.py`. To solve this issue you can apply `Metho-3:` Custom Global Pagination


## Method-2: Per-View Pagination
If you want pagination only for one endpoint (MaterialViewSet):

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

### Method-3: Custom Global Pagination
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

### ✅ Method-1,2,3 Works Automatically For

All **DRF views that inherit from `GenericAPIView`**, including:

| View Type | Works Automatically? |
| --- | --- |
| `ModelViewSet` / `ReadOnlyModelViewSet` | ✅ Yes |
| `ListAPIView` | ✅ Yes |
| `ListCreateAPIView` | ✅ Yes |
| Any view inheriting `GenericAPIView` + `ListModelMixin` | ✅ Yes |



### 🚫 Method-1,2,3 Does NOT Work Automatically For

| View Type | Requires Manual Pagination? |
| --- | --- |
| Plain `APIView` | ❌ Yes |
| Function-Based Views (`@api_view`) | ❌ Yes |
| Any custom view that returns a Response directly without using `GenericAPIView` | ❌ Yes |
- For these, you must manually create a paginator, call `paginate_queryset()`, and return `get_paginated_response()`.
