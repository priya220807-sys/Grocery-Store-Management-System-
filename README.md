# Grocery Store Management System – REST API (Python)

A Django REST Framework backend based on the Month 6 project brief.

## Features
- Customer registration and JWT login
- Role-based access: customer, admin and delivery staff
- Grocery item CRUD (read is public; create/update/delete are admin-only)
- Orders, order history, order detail and staff/admin status updates
- Stock validation and automatic stock reduction when an order is placed
- Search, category/brand filters, ordering and pagination
- Swagger API documentation
- Automated API tests

## 1. Requirements
- Python 3.10+
- pip
- VS Code (recommended)
- SQLite for easiest local setup, or PostgreSQL

## 2. Setup (Windows PowerShell)
```powershell
py -m venv venv
.env\Scripts\Activate.ps1
pip install -r requirements.txt
Copy-Item .env.example .env
py manage.py makemigrations store
py manage.py migrate
py manage.py createsuperuser
py manage.py runserver
```
Open:
- API root health check: http://127.0.0.1:8000/api/health/
- Swagger docs: http://127.0.0.1:8000/api/docs/
- Django admin: http://127.0.0.1:8000/admin/

For Command Prompt activation use `venv\Scripts\activate.bat`.

## 3. Create an admin account
Public `/api/register/` intentionally creates customer accounts only. To create an admin:
1. Run `py manage.py createsuperuser`, then use Django admin; or
2. In Django shell:
```python
from store.models import User
u = User.objects.create_user(username="storeadmin", email="admin@example.com",
    password="StrongPass123!", full_name="Store Admin", role="admin")
```
Never expose admin registration publicly.

## 4. Main API endpoints
| Method | Endpoint | Purpose | Access |
|---|---|---|---|
| GET | `/api/items/` | List items | Public |
| GET | `/api/items/{id}/` | Item detail | Public |
| POST | `/api/items/` | Add item | Admin |
| PUT/PATCH | `/api/items/{id}/` | Update item | Admin |
| DELETE | `/api/items/{id}/` | Delete item | Admin |
| POST | `/api/register/` | Register customer | Public |
| POST | `/api/login/` | Obtain JWT access/refresh tokens | Public |
| POST | `/api/token/refresh/` | Refresh access token | Public |
| GET | `/api/orders/` | Admin sees all; customer sees own | Authenticated |
| GET | `/api/orders/user/` | Current user's order history | Authenticated |
| GET | `/api/orders/{id}/` | Order detail | Owner/Admin/Staff |
| POST | `/api/orders/` | Place order | Customer |
| PUT/PATCH | `/api/orders/{id}/status/` | Update status | Admin/Staff |

Note: the project brief writes paths without trailing slashes in places. Django's default URL configuration here uses trailing slashes.

## 5. Example requests

### Register customer
`POST /api/register/`
```json
{
  "username": "riya",
  "full_name": "Riya Sharma",
  "email": "riya@example.com",
  "password": "StrongPass123!",
  "role": "customer"
}
```

### Login
`POST /api/login/`
```json
{"username": "riya", "password": "StrongPass123!"}
```
Copy the returned `access` token. In Postman set Authorization type to Bearer Token and paste it.

### Create item (admin)
`POST /api/items/`
```json
{
  "name": "Fresh Apples",
  "category": "Fruits",
  "brand": "Farm Fresh",
  "price": "120.00",
  "description": "Fresh red apples",
  "stock_quantity": 50,
  "image_url": "https://example.com/apple.jpg",
  "discount": "5.00",
  "expiry_date": null
}
```

### Place order (customer)
`POST /api/orders/`
```json
{
  "delivery_address": "Jaipur, Rajasthan",
  "payment_method": "Cash on Delivery",
  "items": [
    {"item_id": 1, "quantity": 2}
  ]
}
```
The API checks available stock, stores the order, calculates the total, and reduces inventory.

### Update order status (admin/staff)
`PATCH /api/orders/1/status/`
```json
{"status": "Dispatched"}
```

## 6. Search, filtering and pagination
- `/api/items/?search=apple` searches name, brand and category.
- `/api/items/?category=Fruits`
- `/api/items/?brand=Farm%20Fresh`
- `/api/items/?ordering=price`
- `/api/items/?page=2`
- `/api/orders/?page=2`

## 7. Run tests
```powershell
py manage.py test
```

## 8. Database
SQLite is configured by default so the project runs locally without extra database installation. To use PostgreSQL:
1. Install and create a PostgreSQL database.
2. In `.env`, set `DB_ENGINE=postgres`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`, `DB_HOST`, `DB_PORT`.
3. Run migrations again.

## 9. Deliverables checklist from the PDF
- [x] Django REST Framework API source code
- [x] JWT authentication
- [x] Swagger documentation
- [x] Database models and relationships
- [x] Search, filtering and pagination
- [x] Error responses through DRF validation/permissions
- [x] Unit test starter suite
- [ ] Execute tests locally and capture screenshots
- [ ] Create/import a Postman collection and test each endpoint
- [ ] Configure PostgreSQL/MySQL if required by your mentor

## Important security note
This is a learning project starter. Before production use, add stronger deployment settings, HTTPS, rate limiting, audit logging, payment-provider verification and more complete tests. Do not commit `.env` or real secrets.
