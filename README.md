# Django Bestpractice

## Table of Contents

1. [Overview of Django Bestpractice](#Overview-of-Django-Bestpractice)

   - [Where Business Logic Should Reside](#Where-Business-Logic-Should-Reside)
   - [Where Business Logic Should Not Reside](#Where-Business-Logic-Should-Not-Reside)
   - [Why Avoid Putting Business Logic in APIs, Views, Serializers, or Forms?](#Why-Avoid-Putting-Business-Logic-in-APIs-Views-Serializers-or-Forms)
   - [Why Avoid Custom Managers and Querysets for Business Logic?](#Why-Avoid-Custom-Managers-and-Querysets-for-Business-Logic)
   - [Why Avoid Signals for Business Logic?](#Why-Avoid-Signals-for-Business-Logic)
   - [Model Properties vs Selectors](#Model-Properties-vs-Selectors)
   - [Additional Recommendations](#Additional-Recommendations)
   - [Principle](#Principle)
   - [Starting with a Cookiecutter Template](#Starting-with-a-Cookiecutter-Template)

2. [Working with Django Models](#Working-with-Django-Models)

   - [Defining a Base Model](#Defining-a-Base-Model)
   - [Adding Validation - `clean` and `full_clean`](#adding-validation---clean-and-full_clean)
   - [Using Django's Constraints for Validation](#using-djangos-constraints-for-validation)
   - [Using Model Properties](#using-model-properties)
   - [Implementing Model Methods](#implementing-model-methods)
   - [Testing Your Models](#testing-your-models)
   - [Final Thoughts on Model Design](#final-thoughts-on-model-design)

3. [Services in Django](#Services-in-Django)

   - [Function-Based Service Example](#Function-Based-Service-Example)
   - [Class-Based Service Example](#Class-Based-Service-Example)
   - [Structuring Services in Django Projects](#Structuring-Services-in-Django-Projects)

4. [APIs & Serializers](#apis--serializers)

   - [General API Guidelines](#general-api-guidelines)
   - [Serialization Guidelines](#serialization-guidelines)
   - [Naming Conventions](#naming-conventions)
   - [Class-based vs. Function-based APIs](#class-based-vs-function-based-apis)
     - [Example: Class-based API](#example-class-based-api)
     - [Example: Function-based API](#example-function-based-api)
   - [List APIs](#list-apis)
     - [Plain List API](#plain-list-api)
     - [Filtered and Paginated List API](#filtered-and-paginated-list-api)
   - [Selector Example](#selector-example)
   - [Pagination Utility](#pagination-utility)
   - [Detail API Example](#detail-api-example)
   - [Create API Example](#create-api-example)
   - [Update API Example](#update-api-example)
   - [Object Fetching Utility](#object-fetching-utility)
   - [Inline Serializer Example](#inline-serializer-example)
   - [Advanced Serialization Example](#advanced-serialization-example)

5. [URLs](#urls)
   - [General Guidelines](#General-Guidelines)
     - [Example: Organized URL Patterns](#Example-Organized-URL-Patterns)
     - [Example: Tree-like URL Structure](#Example-Tree-like-URL-Structure)
     - [Example: More Comprehensive URL Patterns](#Example-More-Comprehensive-URL-patterns)
6. [Settings](#Settings)

   - [Folder Structure](#Folder-Structure)
   - [Django-Specific Settings](#Django-Specific-Settings)
   - [Other Settings](#Other-Settings)
   - [Environment Variables](#Environment-Variables)

7. [Integrations](#Integrations)

   - [Example: Integration-Specific Settings](#Example-Integration-Specific-Settings)
   - [Reading from `.env`](#Reading-from-env)

8. [Error and Exception Handling](#Error-and-Exception-Handling)
   - [General Guidelines for Error Handeling](#General-Guidelines-for-Error-Handeling)
   - [Handling Django and DRF Exceptions](#Handling-Django-and-DRF-Exceptions)
   - [Proposed Approach](#Proposed-Approach)
   - [Custom Exception Handlers](#Custom-Exception-Handlers)
   - [More Ideas](#More-Ideas)
9. [Testing](#Testing) - [General Guidelines for Testing](#General-Guidelines-for-Testing) - [Testing with Django and DRF](#Testing-with-Django-and-DRF) - [Using Faker for Data Generation](#Using-Faker-for-Data-Generation) - [Factories for Advanced Testing](#Factories-for-Advanced-Testing) - [Example: Advanced Usage](#Example-Advanced-Usage)
   <br>

## Overview of Django Bestpractice

The core principles of structuring a Django project can be outlined as follows:

### Where Business Logic Should Reside:

- **Services**: Functions designed primarily for database write operations, such as creating or updating records.
- **Selectors**: Functions that focus on retrieving data from the database, often involving complex queries or filtering.
- **Model Properties**: For straightforward logic related to individual fields, provided they do not introduce significant complexity.
- **Model Clean Methods**: For performing additional validation that goes beyond field-specific checks, with some exceptions.

### Where Business Logic Should Not Reside:

- **APIs and Views**: These should handle request and response processing, leaving business logic to services and selectors.
- **Serializers and Forms**: These should be used for data validation, transformation, and serialization, not business rules.
- **Form Tags**: These are meant for rendering HTML forms and should not contain business logic.
- **Model Save Methods**: Custom save logic should be minimized to keep the model simple and focused on representing data.
- **Querysets**: These should primarily deal with query operations and not be overloaded with business logic.
- **Signals**: Signals can lead to hidden dependencies and make the code harder to maintain and debug.

### Why Avoid Putting Business Logic in APIs, Views, Serializers, or Forms?

Using APIs, Views, serializers, and forms for business logic can:

- **Fragment the Logic**: Spreading business logic across multiple layers makes it difficult to trace the flow of data and understand the application’s behavior.
- **Hide Details**: You may need to delve into the inner workings of these abstractions to make changes, leading to increased complexity and potential errors.

While these components are excellent for simple CRUD operations, real-world applications often involve more complex logic, necessitating a more organized approach.

### Why Avoid Custom Managers and Querysets for Business Logic?

Custom managers and querysets can be beneficial for exposing better APIs tailored to your domain, but they are not ideal for encapsulating all business logic because:

- **Domain Logic**: Business logic often spans multiple models and doesn’t always map directly to the data model.
- **Cross-Model Operations**: Logic that involves multiple models can be challenging to place within a single manager or queryset.
- **External Dependencies**: Business logic might require interactions with external systems, which should be kept out of custom manager methods.

Instead, separating domain logic into a service layer ensures that your application remains modular and maintainable.

### Why Avoid Signals for Business Logic?

Using signals to manage business logic can quickly lead to issues:

- **Implicit Connections**: Signals are meant to decouple components, but using them for closely related logic makes these connections implicit and harder to trace.
- **Cache Invalidation**: While signals can handle cache invalidation well, they should not manage core business logic due to the complexity they introduce.

Reserve signals for specific use cases where decoupling is essential, but avoid them for the primary domain or business logic.

### Model Properties vs Selectors:

- **Complex Relationships**: If a property involves traversing multiple relationships, it is better implemented as a selector.
- **Performance Concerns**: If a property is complex and likely to cause N + 1 query issues when serialized, it should be a selector to ensure efficient data retrieval.

### Additional Recommendations:

- **Separation of Concerns**: Ensure that each component of the application has a distinct responsibility, improving maintainability and testability.
- **Reusability**: Design services and selectors to be reusable across different parts of the application, promoting DRY (Don't Repeat Yourself) principles.
- **Testing**: Isolate business logic in services and selectors to facilitate unit testing without involving the entire application stack.
- **Documentation**: Clearly document the purpose and usage of each service and selector to aid understanding and collaboration among team members.

### Principle:

The overarching principle is to "separate concerns" to enhance maintainability, testability, and clarity within the Django project.

### Starting with a Cookiecutter Template

To set up a project with a solid structure from the beginning, consider using a cookiecutter template. Here are a few recommendations:

- **Styleguide-Example**: Use this project as a starting point.
- **cookiecutter-django**: This template includes a wide range of useful features.
- **Custom Cookiecutter**: Create a custom template that suits your specific needs and turn it into a reusable cookiecutter project.

## Working with Django Models

Django models should primarily manage the data model and avoid business logic as much as possible.

### Defining a Base Model

Creating a `BaseModel` to inherit from is a good practice. This base model can include common fields like `created_at` and `updated_at`. Here’s an example:

```python
from django.db import models
from django.utils import timezone

class BaseModel(models.Model):
    created_at = models.DateTimeField(db_index=True, default=timezone.now)
    updated_at = models.DateTimeField(auto_now=True)

    class Meta:
        abstract = True
```

Inheriting from `BaseModel` ensures these fields are included in all your models:

```python
class SomeModel(BaseModel):
    pass
```

### Adding Validation - `clean` and `full_clean`

For additional validation, use the model's `clean` method. This method ensures that data meets certain criteria before being saved to the database. For example:

```python
class Course(BaseModel):
    name = models.CharField(unique=True, max_length=255)
    start_date = models.DateField()
    end_date = models.DateField()

    def clean(self):
        if self.start_date >= self.end_date:
            raise ValidationError("End date cannot be before start date")
```

To invoke the `clean` method, call `full_clean` before saving an instance:

```python
def course_create(*, name: str, start_date: date, end_date: date) -> Course:
    obj = Course(name=name, start_date=start_date, end_date=end_date)
    obj.full_clean()
    obj.save()
    return obj
```

This approach works well with Django admin, as the forms there trigger `full_clean` on the instance.

**General Rules for Using clean**

- Use clean for validation based on multiple non-relational fields of the model.
- Move complex validation or validation involving multiple models to the service layer.

### Using Django's Constraints for Validation

Leverage Django's model constraints to enforce validation rules at the database level. This reduces the amount of code you need to write and maintain. Here's an example:

```python
class Course(BaseModel):
    name = models.CharField(unique=True, max_length=255)
    start_date = models.DateField()
    end_date = models.DateField()

    class Meta:
        constraints = [
            models.CheckConstraint(
                name="start_date_before_end_date",
                check=Q(start_date__lt=F("end_date"))
            )
        ]
```

Django 4.1 and later calls `full_clean` to check model constraints, raising `ValidationError` instead of `IntegrityError`.

For more examples on using constraints, refer to these articles by Adam Johnson:

- [Using Django Check Constraints to Ensure Only One Field Is Set](https://adamj.eu/tech/2020/03/25/django-check-constraints-one-field-set/)
- [Django’s Field Choices Don’t Constrain Your Data](https://adamj.eu/tech/2020/01/22/djangos-field-choices-dont-constrain-your-data/)
- [Using Django Check Constraints to Prevent Self-Following](https://adamj.eu/tech/2021/02/26/django-check-constraints-prevent-self-following/)

### Using Model Properties

Model properties allow easy access to derived values from a model instance. For example:

```python
from django.utils import timezone

class Course(BaseModel):
    name = models.CharField(unique=True, max_length=255)
    start_date = models.DateField()
    end_date = models.DateField()

    @property
    def has_started(self) -> bool:
        return self.start_date <= timezone.now().date()

    @property
    def has_finished(self) -> bool:
        return self.end_date <= timezone.now().date()
```

Use properties for simple derived values based on non-relational fields. For more complex logic or values involving multiple relations, use services or selectors.

### Implementing Model Methods

Model methods are useful for more complex operations that require arguments or setting multiple fields. For example:

```python
class Course(BaseModel):
    name = models.CharField(unique=True, max_length=255)
    start_date = models.DateField()
    end_date = models.DateField()

    def is_within(self, date: date) -> bool:
        return self.start_date <= date <= self.end_date
```

For attribute setting that involves multiple fields:

```python
from django.utils.crypto import get_random_string
from django.conf import settings
from django.utils import timezone

class Token(BaseModel):
    secret = models.CharField(max_length=255, unique=True)
    expiry = models.DateTimeField(blank=True, null=True)

    def set_new_secret(self):
        now = timezone.now()
        self.secret = get_random_string(255)
        self.expiry is now + settings.TOKEN_EXPIRY_TIMEDELTA
        return self
```

**General Guidelines for Model Methods**

- Use methods for derived values requiring arguments.
- Use methods for setting related fields together.
- For complex logic or spanning multiple relations, prefer services or selectors.

### Testing Your Models

Models should be tested, especially if they contain custom validation, properties, or methods. Example test:

```python
from datetime import timedelta
from django.test import TestCase
from django.core.exceptions import ValidationError
from django.utils import timezone
from project.some_app.models import Course

class CourseTests(TestCase):
    def test_course_end_date_cannot_be_before_start_date(self):
        start_date = timezone.now()
        end_date = timezone.now() - timedelta(days=1)
        course = Course(start_date=start_date, end_date=end_date)

        with self.assertRaises(ValidationError):
            course.full_clean()
```

In this test, we assert that a `ValidationError` is raised without hitting the database, which speeds up the tests.

### Final Thoughts on Model Design

Keep the following in mind for effective model design:

- Keep models simple and focused on the data model.
- Use services, selectors, and utilities for complex logic.
- Validate data at the right layer: model, service, or database constraints.
- Write comprehensive tests for any custom logic within models.

## Services in Django

Services in Django encapsulate business logic, ensuring separation of concerns and promoting reusability and maintainability. They can be implemented as functions or classes, depending on the complexity and structure of your application.

### Function-Based Service Example

Here's an example of a function-based service for creating a user:

```python
def user_create(
    *,
    email: str,
    name: str
) -> User:
    user = User(email=email)
    user.full_clean()
    user.save()

    profile_create(user=user, name=name)
    send_confirmation_email(user=user)

    return user
```

This service function demonstrates the creation of a user and calls other services (`profile_create` and `send_confirmation_email`) to complete the process.

### Class-Based Service Example

Class-based services provide a structured approach, particularly useful for complex business logic. Here's an example:

```python
from django.contrib.auth.models import User
from django.core.exceptions import ValidationError
from django.db import transaction

class UserService:
    """
    Service class for user-related operations.
    """

    @transaction.atomic
    def create_user(self, email: str, username: str, password: str) -> User:
        """
        Creates a new user with the provided email, username, and password.
        """
        user = User(email=email, username=username)
        user.set_password(password)
        user.full_clean()
        user.save()
        return user

    @transaction.atomic
    def update_user(self, user: User, email: str, username: str) -> User:
        """
        Updates the email and username of the given user instance.
        """
        user.email = email
        user.username = username
        user.full_clean()
        user.save()
        return user

    @transaction.atomic
    def delete_user(self, user: User):
        """
        Deletes the given user instance.
        """
        user.delete()
```

This UserService class provides methods to create, update, and delete users, ensuring transactions for data integrity.

### Using Services in Views

Services are used within views to execute business logic. Here’s how you might use UserService in Django views:

```python
from django.shortcuts import get_object_or_404
from django.http import JsonResponse
from .services import UserService

def create_user_view(request):
    if request.method == 'POST':
        email = request.POST.get('email')
        username = request.POST.get('username')
        password = request.POST.get('password')

        user_service = UserService()
        try:
            new_user = user_service.create_user(email=email, username=username, password=password)
            return JsonResponse({'message': 'User created successfully', 'user_id': new_user.id}, status=201)
        except ValidationError as e:
            return JsonResponse({'error': str(e)}, status=400)

def update_user_view(request, user_id):
    user = get_object_or_404(User, id=user_id)
    if request.method == 'PUT':
        email = request.PUT.get('email')
        username = request.PUT.get('username')

        user_service = UserService()
        try:
            updated_user = user_service.update_user(user=user, email=email, username=username)
            return JsonResponse({'message': 'User updated successfully', 'user_id': updated_user.id})
        except ValidationError as e:
            return JsonResponse({'error': str(e)}, status=400)

def delete_user_view(request, user_id):
    user = get_object_or_404(User, id=user_id)
    if request.method == 'DELETE':
        user_service = UserService()
        user_service.delete_user(user=user)
        return JsonResponse({'message': 'User deleted successfully'}, status=204)
```

### Structuring Services in Django Projects

In larger projects, organizing services into modules or sub-modules improves maintainability. Example structure:

    project/
    ├── your_app/
    │   ├── services/
    │   │   ├── __init__.py
    │   │   ├── user_services.py
    │   │   ├── file_services.py
    │   │   └── ...
    │   ├── views.py
    │   ├── models.py
    │   └── ...
    └── ...

Each service module (`user_services.py`, `file_services.py`, etc.) contains related business logic, enhancing code organization and scalability.

### Conclusion

Services in Django centralize business logic, ensuring clear separation of concerns and facilitating code reusability. Whether function-based or class-based, structuring services systematically enhances project maintainability and scalability.

## APIs & Serializers

When working with services and selectors, aim for consistency and simplicity in your APIs.

### General API Guidelines

1. One API per operation: Create separate APIs for each CRUD operation.
2. Inherit from simple views: Use APIView or GenericAPIView to manage logic through services and selectors, not serializers.
3. No business logic in APIs: Keep business logic out of the API. Object fetching and data manipulation can be done within the API or extracted to service functions.
4. Simplicity: Keep APIs straightforward as they serve as interfaces to your core business logic.

### Serialization Guidelines

1. Separate input and output serializers:

- Input serializers handle incoming data.
- Output serializers manage outgoing data.

2. Use appropriate abstractions: Choose the serialization method that fits your needs.
3. DRF Serializers:

- Nest serializers within the API and name them InputSerializer or OutputSerializer.
- Prefer Serializer over ModelSerializer unless the latter fits your use case better.
- Use inline serializers for nested data.
- Minimize serializer reuse to avoid unexpected behavior.

### Naming Conventions

Follow the pattern: `<Entity><Action>Api`. Examples: `ProductCreateApi`, `ProductUpdateApi`.

### Class-based vs. Function-based APIs

- Default to class-based: They allow inheritance and attribute/method nesting, facilitating mixins and additional configurations.
- Function-based: If preferred, use decorators to achieve similar results.

#### Example: Class-based API

```python
class ProductDetailApi(BaseApi):
    def get(self, request):
        data = fetch_product_details()
        return Response(data)
```

#### Example: Function-based API

```python
@base_api(["GET"])
def product_detail_api(request):
    data = fetch_product_details()
    return Response(data)
```

### List APIs

#### Plain List API

```python
from rest_framework.views import APIView
from rest_framework import serializers
from rest_framework.response import Response

from ecommerce.products.selectors import product_list

class ProductListApi(APIView):
    class OutputSerializer(serializers.Serializer):
        id = serializers.CharField()
        name = serializers.CharField()

    def get(self, request):
        products = product_list()
        data = self.OutputSerializer(products, many=True).data
        return Response(data)
```

#### Filtered and Paginated List API

```python
from rest_framework.views import APIView
from rest_framework import serializers
from ecommerce.api.pagination import get_paginated_response, LimitOffsetPagination
from ecommerce.products.selectors import product_list

class ProductListApi(APIView):
    class Pagination(LimitOffsetPagination):
        default_limit = 5

    class FilterSerializer(serializers.Serializer):
        id = serializers.IntegerField(required=False)
        category = serializers.CharField(required=False)
        price_min = serializers.DecimalField(required=False, max_digits=10, decimal_places=2)
        price_max = serializers.DecimalField(required=False, max_digits=10, decimal_places=2)

    class OutputSerializer(serializers.Serializer):
        id = serializers.CharField()
        name = serializers.CharField()
        price = serializers.DecimalField(max_digits=10, decimal_places=2)

    def get(self, request):
        filters_serializer = self.FilterSerializer(data=request.query_params)
        filters_serializer.is_valid(raise_exception=True)

        products = product_list(filters=filters_serializer.validated_data)
        return get_paginated_response(
            pagination_class=self.Pagination,
            serializer_class=self.OutputSerializer,
            queryset=products,
            request=request,
            view=self
        )
```

### Selector Example

```python
import django_filters
from ecommerce.products.models import Product

class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = ('id', 'category', 'price')

def product_list(*, filters=None):
    filters = filters or {}
    qs = Product.objects.all()
    return ProductFilter(filters, qs).qs
```

### Pagination Utility

```python
from rest_framework.response import Response

def get_paginated_response(*, pagination_class, serializer_class, queryset, request, view):
    paginator = pagination_class()
    page = paginator.paginate_queryset(queryset, request, view=view)
    if page is not None:
        serializer = serializer_class(page, many=True)
        return paginator.get_paginated_response(serializer.data)
    serializer = serializer_class(queryset, many=True)
    return Response(data=serializer.data)
```

### Detail API Example

```python
class ProductDetailApi(SomeAuthenticationMixin, APIView):
    class OutputSerializer(serializers.Serializer):
        id = serializers.CharField()
        name = serializers.CharField()
        price = serializers.DecimalField(max_digits=10, decimal_places=2)
        description = serializers.CharField()

    def get(self, request, product_id):
        product = fetch_product(id=product_id)
        serializer = self.OutputSerializer(product)
        return Response(serializer.data)
```

### Create API Example

```python
class ProductCreateApi(SomeAuthenticationMixin, APIView):
    class InputSerializer(serializers.Serializer):
        name = serializers.CharField()
        price = serializers.DecimalField(max_digits=10, decimal_places=2)
        description = serializers.CharField()

    def post(self, request):
        serializer = self.InputSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        create_product(**serializer.validated_data)
        return Response(status=status.HTTP_201_CREATED)
```

### Update API Example

```python
class ProductUpdateApi(SomeAuthenticationMixin, APIView):
    class InputSerializer(serializers.Serializer):
        name = serializers.CharField(required=False)
        price = serializers.DecimalField(required=False, max_digits=10, decimal_places=2)
        description = serializers.CharField(required=False)

    def post(self, request, product_id):
        serializer = self.InputSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        update_product(product_id=product_id, **serializer.validated_data)
        return Response(status=status.HTTP_200_OK)
```

### Object Fetching Utility

```python
from django.shortcuts import get_object_or_404
from django.http import Http404

def get_object(model_or_queryset, **kwargs):
    try:
        return get_object_or_404(model_or_queryset, **kwargs)
    except Http404:
        return None
```

### Inline Serializer Example

```python
class Serializer(serializers.Serializer):
    categories = inline_serializer(many=True, fields={
        'id': serializers.IntegerField(),
        'name': serializers.CharField(),
    })
```

### Advanced Serialization Example

```python
class ProductFeedApi(BaseApi):
    def get(self, request):
        feed = fetch_product_feed(user=request.user)
        data = serialize_product_feed(feed)
        return Response(data)

class FeedItemSerializer(serializers.Serializer):
    id = serializers.CharField()
    name = serializers.CharField()
    price = serializers.DecimalField(max_digits=10, decimal_places=2)
    calculated_discount = serializers.DecimalField(max_digits=10, decimal_places=2, source="_calculated_discount")

def serialize_product_feed(feed):
    feed_ids = [item.id for item in feed]
    objects = Product.objects.filter(id__in=feed_ids).order_by("-created_at")
    discount_cache = get_discount_cache(feed_ids)

    result = []
    for item in objects:
        item._calculated_discount = discount_cache.get(item.id)
        result.append(FeedItemSerializer(item).data)

    return result
```

## URLs

We recommend organizing URLs similarly to how APIs are organized, with one URL per action. This ensures clarity and ease of maintenance.

### General Guidelines

- Separate URL patterns by domain: Split URLs for different domains into their own lists and include them from urlpatterns.
- Single URL per action: Create a distinct URL for each CRUD operation.

#### Example: Organized URL Patterns

```python
from django.urls import path, include
from myapp.apis import (
    ItemCreateApi,
    ItemUpdateApi,
    ItemListApi,
    ItemDetailApi,
    ItemCustomActionApi,
)

item_patterns = [
    path('', ItemListApi.as_view(), name='list'),
    path('<int:item_id>/', ItemDetailApi.as_view(), name='detail'),
    path('create/', ItemCreateApi.as_view(), name='create'),
    path('<int:item_id>/update/', ItemUpdateApi.as_view(), name='update'),
    path('<int:item_id>/custom-action/', ItemCustomActionApi.as_view(), name='custom-action'),
]

urlpatterns = [
    path('items/', include((item_patterns, 'items'))),
]
```

#### Example: Tree-like URL Structure

```python
from django.urls import path, include
from example_project.files.apis import (
    FileDirectUploadApi,
    FileTransferUploadStartApi,
    FileTransferUploadFinishApi,
    FileTransferUploadLocalApi,
)

urlpatterns = [
    path(
        "upload/",
        include(([
            path("direct/", FileDirectUploadApi.as_view(), name="direct"),
            path(
                "transfer/",
                include(([
                    path("start/", FileTransferUploadStartApi.as_view(), name="start"),
                    path("finish/", FileTransferUploadFinishApi.as_view(), name="finish"),
                    path("local/<str:file_id>/", FileTransferUploadLocalApi.as_view(), name="local"),
                ], "transfer"))
            ),
        ], "upload"))
    )
]
```

#### Example: More Comprehensive URL Patterns

For larger projects, keeping the URLs well-organized is crucial. Here’s an extended example:

```python
from django.urls import path, include
from myproject.orders.apis import (
    OrderCreateApi,
    OrderListApi,
    OrderDetailApi,
    OrderUpdateApi,
    OrderDeleteApi,
)

order_patterns = [
    path('', OrderListApi.as_view(), name='list'),
    path('create/', OrderCreateApi.as_view(), name='create'),
    path('<int:order_id>/', OrderDetailApi.as_view(), name='detail'),
    path('<int:order_id>/update/', OrderUpdateApi.as_view(), name='update'),
    path('<int:order_id>/delete/', OrderDeleteApi.as_view(), name='delete'),
]

urlpatterns = [
    path('orders/', include((order_patterns, 'orders'))),
]
```

## Settings

Following a structured approach to settings helps in managing different environments (development, production, etc.) effectively.

### Folder Structure

We follow a folder structure inspired by the cookiecutter-django template, with some adjustments:

    config/
    ├── __init__.py
    ├── django/
    │   ├── __init__.py
    │   ├── base.py
    │   ├── local.py
    │   ├── production.py
    │   └── test.py
    ├── settings/
    │   ├── __init__.py
    │   ├── celery.py
    │   ├── cors.py
    │   ├── sentry.py
    │   └── sessions.py
    ├── urls.py
    ├── env.py
    └── wsgi.py
    └── asgi.py

### Django-Specific Settings

In the config/django directory, we place all Django-related settings:

- `base.py `contains the bulk of the settings and imports additional settings from config/settings.
- `production.py` imports from `base.py` and overrides settings specific to production.
- `test.py` imports from `base.py` and adjusts settings for testing.
- `local.py` imports from `base.py` and can override settings for local development.

### Other Settings

In the config/settings directory, we place additional settings:

- Celery configuration
- Third-party service configurations (e.g., CORS, Sentry)

### Environment Variables

We often use a `config/env.py` file for handling environment variables:

```python
import environ

env = environ.Env()

# Usage in base.py
from config.env import env

# Reading environment variables
DEBUG = env.bool('DEBUG', default=False)
ALLOWED_HOSTS = env.list('ALLOWED_HOSTS', default=[])
```

#### Prefixing Environment Variables

We generally prefix Django-specific environment variables with `DJANGO_` to distinguish them from other applications running in the same environment.
For example:

```python
# Prefixed variables
DJANGO_SETTINGS_MODULE
DJANGO_DEBUG
DJANGO_ALLOWED_HOSTS

# Non-prefixed variables
AWS_SECRET_KEY
CELERY_BROKER_URL
EMAILS_ENABLED
```

## Integrations

### Example: Integration-Specific Settings

Integration-specific settings are placed in `config/settings/<integration>.py` with a flag to enable or disable the integration based on environment variables.

```python
from config.env import env

USE_SENTRY = env.bool('USE_SENTRY', default=False)

if USE_SENTRY:
    SENTRY_DSN = env('SENTRY_DSN')
    import sentry_sdk
    from sentry_sdk.integrations.django import DjangoIntegration

    sentry_sdk.init(
        dsn=SENTRY_DSN,
        integrations=[DjangoIntegration()],
    )
```

### Reading from `.env`

Using a `.env` file for local development is a common practice, facilitated by django-environ:

```python
# In base.py
import os
from config.env import env, environ

BASE_DIR = environ.Path(__file__) - 3
env.read_env(os.path.join(BASE_DIR, ".env"))
```

Make sure to exclude `.env` from source control and provide a `.env.example` with sample values for new developers.

```ini
# .env.example
DJANGO_DEBUG=True
DJANGO_ALLOWED_HOSTS=localhost,127.0.0.1
AWS_SECRET_KEY=
CELERY_BROKER_URL=
EMAILS_ENABLED=True
```

## Error and Exception Handling

Handling errors and exceptions is crucial for robust API design. Here are some guidelines and approaches to handle them effectively in Django REST Framework (DRF).

### General Guidelines for Error Handeling

- Understand how DRF handles exceptions.
- Define the structure of your API error responses.
- Customize exception handling to suit your needs.

### Handling Django and DRF Exceptions

**DRF's `ValidationError`**

```python
from rest_framework.exceptions import ValidationError

def some_service():
    raise ValidationError("Invalid data")
```

Response:

```json
["Invalid data"]
```

Or:

```python
def some_service():
    raise ValidationError({"error": "Invalid data"})
```

Response:

```json
{
  "error": "Invalid data"
}
```

Django's `ValidationError`

DRF doesn't handle Django's `ValidationError` by default:

```python
from django.core.exceptions import ValidationError as DjangoValidationError

def some_service():
    raise DjangoValidationError("Invalid input")
```

To handle this, you need a custom exception handler:

```python
from django.core.exceptions import ValidationError as DjangoValidationError
from rest_framework.views import exception_handler
from rest_framework.exceptions import ValidationError as DRFValidationError
from rest_framework.serializers import as_serializer_error

def custom_exception_handler(exc, context):
    if isinstance(exc, DjangoValidationError):
        exc = DRFValidationError(as_serializer_error(exc))

    response = exception_handler(exc, context)
    return response
```

### Proposed Approach

We propose a consistent structure for error responses:

```json
{
  "message": "Error message",
  "details": {}
}
```

### Custom Exception Handlers

```python
from django.core.exceptions import ValidationError as DjangoValidationError, PermissionDenied
from django.http import Http404
from rest_framework.views import exception_handler
from rest_framework.exceptions import ValidationError as DRFValidationError
from rest_framework.response import Response
from myapp.core.exceptions import CustomApplicationError

def custom_exception_handler(exc, context):
    if isinstance(exc, DjangoValidationError):
        exc = DRFValidationError(as_serializer_error(exc))

    if isinstance(exc, Http404):
        exc = DRFValidationError("Not found")

    if isinstance(exc, PermissionDenied):
        exc = DRFValidationError("Permission denied")

    response = exception_handler(exc, context)

    if response is None:
        if isinstance(exc, CustomApplicationError):
            return Response({
                "message": exc.message,
                "details": exc.details
            }, status=400)
        return response

    if isinstance(exc.detail, (list, dict)):
        response.data = {
            "details": response.data
        }
    response.data["message"] = "Error occurred"
    return response
```

### More Ideas

- Extending Exception Hierarchy: Define custom exceptions like `CustomValidationError` and `CustomPermissionError`.
- Consistent Logging: Ensure all exceptions are logged properly, perhaps by integrating with a service like Sentry.
- Translation: Convert `django.core.exceptions.ObjectDoesNotExist` to `rest_framework.exceptions.NotFound`.

Here's an example of integrating `ObjectDoesNotExist` handling:

```python
from django.core.exceptions import ObjectDoesNotExist

def custom_exception_handler(exc, context):
    if isinstance(exc, ObjectDoesNotExist):
        exc = DRFValidationError("Object not found")

    response = exception_handler(exc, context)
    return response
```

By following these guidelines and examples, you can create a robust and consistent error handling mechanism for your Django applications.

## Testing

### General Guidelines for Testing

Testing is essential for ensuring the reliability and stability of your Django and Django REST Framework (DRF) applications. Here are some general guidelines:

- Write tests for every new feature or bug fix.
- Ensure your tests are isolated and repeatable.
- Use factories and Faker to generate test data.

### Testing with Django and DRF

1. Setting Up Your Test Environment

To set up your test environment, make sure you have the necessary dependencies installed:

```bash
pip install pytest pytest-django pytest-factoryboy
```

Add the following to your `pytest.ini`:

```ini
[pytest]
DJANGO_SETTINGS_MODULE = myproject.settings
python_files = tests.py test_*.py *_tests.py
```

3. Writing Basic Tests

Here's an example of a basic test case using Django's built-in test framework:

```python
from django.test import TestCase
from myapp.models import MyModel

class MyModelTestCase(TestCase):
    def setUp(self):
        self.instance = MyModel.objects.create(name="Test")

    def test_instance_creation(self):
        self.assertEqual(self.instance.name, "Test")
```

### Using Faker for Data Generation

1. Introduction to `Faker`
   `Faker` is a Python library that generates fake data. It is very useful for testing purposes where you need realistic but fake data.

Install `Faker` with:

```sh
pip install Faker
```

2. Using `Faker` in Tests

Here's how you can use Faker in your tests:

```python
from faker import Faker
import pytest

fake = Faker()

@pytest.mark.django_db
def test_create_fake_user():
    name = fake.name()
    email = fake.email()

    user = MyModel.objects.create(name=name, email=email)

    assert user.name == name
    assert user.email == email
```

### Factories for Advanced Testing

1. Introduction to Factories
   Factories help you create instances of your models with predefined data. This is particularly useful for setting up tests that require specific states of your database.
2. Implementing Factories

- First, install `factory_boy`:

  ```sh
  pip install factory_boy
  ```

- Then define your factories:

```python
import factory
from myapp.models import MyModel

class MyModelFactory(factory.django.DjangoModelFactory):
    class Meta:
        model = MyModel

    name = factory.Faker('name')
    email = factory.Faker('email')
```

### Example: Advanced Usage

1. Combining Faker and Factories

Combining Faker and factories allows for powerful test setups. Here's an example:

```python
import pytest
from myapp.models import MyModel
from .factories import MyModelFactory

@pytest.mark.django_db
def test_my_model_creation():
    instance = MyModelFactory.create()

    assert isinstance(instance.name, str)
    assert isinstance(instance.email, str)
```

2. Blog Reference
   For more advanced usage and examples, refer to the blog post on HackSoft:

- [Improve Your Tests: Django Fakes and Factories - Advanced Usage](https://www.hacksoft.io/blog/improve-your-tests-django-fakes-and-factories-advanced-usage)
- [DjangoCon 2022 | factory_boy: testing like a pro](https://www.youtube.com/watch?v=-C-XNHAJF-c)

[Back to Top](#Django-Bestpractice)
