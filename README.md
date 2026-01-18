# DCRM – Django CRM Application

DCRM is a lightweight customer management application built with Django and backed by a MySQL database.  
Users can register, authenticate and manage customer records via a clean web-based UI.

---

## 🚀 Features

- User registration & authentication
- CRUD operations for customer records
- MySQL database backend
- Django ORM-based queries
- Protected views (only authenticated users)
- Bootstrap-based UI components
- Server-side form validation
- Secure session handling

---

## 🧰 Tech Stack

**Backend**

- Python 3.x
- Django (Latest)
- Django ORM
- MySQL

**Frontend**

- HTML
- CSS (Bootstrap)
- Django Templates

**Database**

- MySQL (local instance)
- `mysqlclient` / `pymysql` drivers supported

---

## 🗄 MySQL Database Setup

Create MySQL database:

```sql
CREATE DATABASE dcrm_db;

Configure settings.py:

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'dcrm_db',
        'USER': 'root',
        'PASSWORD': '<your-password>',
        'HOST': 'localhost',
        'PORT': '3306',
    }
}


Install dependencies:

pip install mysqlclient
# Or alternative
pip install pymysql


Run migrations:

python manage.py migrate


🏗 Run & Development

Start server:

python manage.py runserver

Visit:

http://127.0.0.1:8000/

🔐 Authentication
Feature	Status
Register	✔
Login	✔
Logout	✔
Protected Views	✔
CRUD Records	✔

Only authenticated users can create, update or delete customer records.

📂 Project Structure (Simplified)
website/
 ├── models.py        # CRM models
 ├── views.py         # CRUD logic
 ├── forms.py         # Django forms
 ├── urls.py          # Routing
 ├── templates/       # HTML templates (Bootstrap)
 └── migrations/


📸 Templates Included

home

register

login

record list

add record

update record

delete (via redirect)

base layout + navbar

📦 Status

This project is not deployed and not intended for production.
It was built purely for learning and hands-on practice with Django + MySQL + Auth + CRUD.

🧮 Next Possible Improvements (Optional)

Pagination

Search filter

REST API via DRF

Unit tests

Dockerization

Deployment (Railway / Render / VPS)

🧑‍💻 Notes

This was my first Django CRUD project using MySQL instead of SQLite.
Additionally, I used this project as an opportunity to structure multiple Git commits and practice using GitHub with clean history.

👨 Author

Ogulcan Erdag
Portfolio → https://ogulcan-erdag.com

GitHub → https://github.com/OgulcanErdag
```
