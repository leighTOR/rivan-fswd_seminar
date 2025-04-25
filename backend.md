## BACKEND INSTALLATION/SETUP

## Task 1: Install Python

```bash
python --version
```

---

## Task 2: Virtual Environment Installation

```bash
python -m venv env
```

---

## Task 3: Feeding Your Project — The Grocery List Edition

Make a new file and name it `requirements.txt`:

```
asgiref
Django
django-cors-headers
djangorestframework
djangorestframework-simplejwt
PyJWT
pytz
sqlparse
psycopg2-binary
python-dotenv
```

In the terminal, paste this:

```bash
pip install -r requirements.txt
```

---

## Task 4: Creating a Django Project

```bash
django-admin startproject backend
cd backend
django-admin startapp api
```

---

## Task 5: Turning Your Django Setup Into a Party!

Configure `settings.py` in the `backend` folder:

```python
from datetime import timedelta
from dotenv import load_dotenv
import os

load_dotenv()

ALLOWED_HOSTS = ["*"]
```

In `INSTALLED_APPS`, add:

```python
‘api’,
‘rest_framework’,
‘corsheaders’,
```

In `MIDDLEWARE`, add:

```python
’corsheaders.middleware.CorsMiddleware’,
```

At the end of the script, add:

```python
CORS_ALLOW_ALL_ORIGINS = True
CORS_ALLOWS_CREDENTIALS = True
```

---

## Task 6: Understanding JWT Tokens – Unlocking the Secret Doors of Your App!

Make a new file inside the `api` folder and name it `serializers.py`, and paste this:

```python
from django.contrib.auth.models import User
from rest_framework import serializers

class UserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ["id", "username", "password"]

    extra_kwargs = {"password": {"write_only": True}}

    def create(self, validated_data):
        print(validated_data)
        user = User.objects.create_user(**validated_data)
        return user
```

---

## Task 7: Let’s Get Those Users Registered!

Configure `views.py`:

```python
from django.shortcuts import render
from django.contrib.auth.models import User
from rest_framework import generics
from .serializers import UserSerializer
from rest_framework.permissions import IsAuthenticated

class CreateUserView(generics.CreateAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
    permission_classes = [AllowAny]
```

---

## Task 8: Set Up the Roads for Our App!

Configure `backend/urls.py`:

```python
from django.contrib import admin
from django.urls import path, include
from api.views import CreateUserView
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/user/register/", CreateUserView.as_view(), name="register"),
    path("api/token/", TokenObtainPairView.as_view(), name="get_token"),
    path("api/token/refresh/", TokenRefreshView.as_view(), name="refresh"),
    path("api-auth/", include("rest_framework.urls")),
]
```

---

## Task 9: Let’s Get This Party Started!

Run your server:

```bash
python manage.py makemigrations
python manage.py migrate
python manage.py runserver
```

---

## Task 10: Navigate to Your URL

- `/api/user/register`
- `/api/user/token`
- `/api/token/refresh`

---

## Task 11: The Blueprint of Your Notes App!

`api/models.py`:

```python
from django.db import models
from django.contrib.auth.models import User

class Note(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    author = models.ForeignKey(User, on_delete=models.CASCADE, related_name="notes")

    def __str__(self):
        return self.title
```

---

## Task 12: Turning Notes into a Work of Art!

Insert these in `serializers.py`:

```python
from .models import Note

class NoteSerializer(serializers.ModelSerializer):
    class Meta:
        model = Note
        fields = ["id", "title", "content", "created_at", "author"]
        extra_kwargs = {"author": {"read_only": True}}
```

---

## Task 13: The Power of Creating and Deleting Notes!

Insert these in `views.py`. Add `NoteSerializer` beside `UserSerializer`:

```python
from .models import Note

class NoteListCreate(generics.ListCreateAPIView):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        user = self.request.user
        return Note.objects.filter(author=user)

    def perform_create(self, serializer):
        if serializer.is_valid():
            serializer.save(author=self.request.user)
        else:
            print(serializer.errors)

class NoteDelete(generics.DestroyAPIView):
    serializer_class = NoteSerializer
    permission_classes = [IsAuthenticated]

    def get_queryset(self):
        user = self.request.user
        return Note.objects.filter(author=user)
```

---

## Task 14: Protecting Your Notes Like a Boss!

Make a new file inside the `api` folder and name it `urls.py`:

```python
from django.urls import path
from . import views

urlpatterns = [
    path("notes/", views.NoteListCreate.as_view(), name="note-list"),
    path("notes/delete/<int:pk>/", views.NoteDelete.as_view(), name="delete-note"),
]
```

---

## Task 15: The Party Starts!

In `backend/urls.py`:

```python
path("api/", include("api.urls")),
```

After this, do **Task 9** again.
