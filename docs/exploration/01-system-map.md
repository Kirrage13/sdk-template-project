# Module 1 — System Map

## 1. System diagram

```mermaid
flowchart TD
    B[Browser]
    F[Frontend - JavaScript / Vite]
    P[Products Service - PHP / Slim]
    U[Users Service - Python / FastAPI]
    O[Orders Service - Java / Spring Boot]
    DB[(PostgreSQL)]
    M[Prisma Migration Runner]

    B -->|localhost:5173| F

    B -->|localhost:8082/products| P
    B -->|localhost:8000/users| U
    B -->|localhost:8083/orders| O

    P -->|database:5432| DB
    U -->|database:5432| DB
    O -->|database:5432| DB

    M -->|migrations and seed data| DB
```

The frontend is opened in the browser on port `5173`.

The JavaScript running in the browser sends requests to the three backend services:

- Products Service — port `8082`
- Users Service — port `8000`
- Orders Service — port `8083`

The backend services connect to PostgreSQL using the service name `database`.

The migration runner prepares the database and then exits.

---

## 2. Request trace — loading products

The request that loads the products on the main page was traced.

### Step 1 — Products API URL

The Products API URL is set in:

`frontend/src/main.js:7`

```js
productsApiUrl: import.meta.env.VITE_PRODUCTS_API_URL || 'http://localhost:8082'
```

With the `.env` file, the Products Service is available at:

```text
http://localhost:8082
```

### Step 2 — Product loading starts

The frontend calls the product rendering function here:

`frontend/src/main.js:55`

```js
renderProducts(config.productsApiUrl)
```

### Step 3 — The `/products` URL is created

Inside `renderProducts`, `/products` is added to the API URL:

`frontend/src/components/products.js:11`

```js
const products = await fetchData(`${apiUrl}/products`)
```

The final request is:

```text
GET http://localhost:8082/products
```

### Step 4 — Browser sends the request

The request is sent using `fetch()`:

`frontend/src/api/api.js:8`

```js
const response = await fetch(url)
```

Docker maps port `8082` on my computer to port `80` inside the Products Service container.

### Step 5 — PHP receives the request

The Products Service has this route:

`products-service/public/index.php:26`

```php
$app->get('/products', function (Request $request, Response $response, $args) {
```

This route handles `GET /products`.

### Step 6 — Connection to PostgreSQL

The database settings are read here:

`products-service/public/index.php:27-31`

The PostgreSQL connection is created here:

`products-service/public/index.php:33-39`

The database host is `database`, because this is the Docker Compose service name.

### Step 7 — Database query

The Products Service reads the products from the `Product` table:

`products-service/public/index.php:41`

```php
$stmt = $pdo->query("SELECT * FROM \"Product\"");
```

The result is read here:

`products-service/public/index.php:42`

```php
$products = $stmt->fetchAll();
```

### Step 8 — PHP returns JSON

The result is converted to JSON:

`products-service/public/index.php:44-45`

```php
$response->getBody()->write(json_encode($products));
return $response->withHeader('Content-Type', 'application/json');
```

### Step 9 — Frontend reads the JSON

The frontend converts the response from JSON:

`frontend/src/api/api.js:12`

```js
return await response.json()
```

### Step 10 — Products are shown on the page

The product cards are created here:

`frontend/src/components/products.js:18-29`

The code uses:

```js
products.map(...)
```

and puts the generated HTML into:

```js
container.innerHTML
```

So the full request path is:

```text
Browser
  ↓
main.js
  ↓
renderProducts()
  ↓
GET http://localhost:8082/products
  ↓
Products Service
  ↓
GET /products route
  ↓
PostgreSQL Product table
  ↓
JSON response
  ↓
Frontend
  ↓
Product cards in the browser
```

---

## 3. Environment gotchas

### Missing `.env` file

On the first start I ran:

```text
docker compose up -d --build
```

but Docker showed warnings like:

```text
The "FRONTEND_PORT" variable is not set.
The "DB_USER" variable is not set.
```

At the end I also got:

```text
invalid proto:
```

The repository had `.env.example`, but there was no `.env` file.

I fixed it with:

```text
copy .env.example .env
```

After that Docker could read the ports and database settings correctly.

The first build took about 500 seconds.

### Migration runner

After starting the system, `docker compose ps -a` showed:

```text
migration-runner   Exited (0)
```

At first this looked like the container had stopped because of a problem.

After checking the project, I understood that this container is supposed to run once, prepare the database, and then exit.

`Exited (0)` means it finished successfully.

---

## 4. Documentation difference

TODO: Will be added later