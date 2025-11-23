# Learning Platform -- REST API dokumentáció (Laravel + Sanctum)

**Base URL:**

    http://127.0.0.1/learningPlatformBearer/public/api

vagy

    http://127.0.0.1:8000/api

Az API célja, hogy egy tanulási platform backendjét biztosítsa, ahol a
tanulók kurzusokra iratkozhatnak be, tananyagokat teljesíthetnek, és
ahol adminisztrátori jogosultsággal a rendszer kezelhető.

------------------------------------------------------------------------

# Funkciók

-   **Authentikáció** (login, token kezelés, logout)
-   **Felhasználó kezelés**
-   **Kurzuskezelés, beiratkozás, teljesítés**
-   **Admin jogosultság** a felhasználók kezeléséhez
-   **Token alapú védelem** (Sanctum)

------------------------------------------------------------------------

# Tesztadatok

-   **1 admin**:

    -   email: `admin@example.com`\
    -   jelszó: `admin`

-   **9 student user** --- jelszó: `Jelszo_2025`

-   **3 kurzus**

-   **2 random student** beiratkozásokkal, köztük 1--1 teljesített

------------------------------------------------------------------------

# Hibaüzenetek

  ------------------------------------------------------------------------
  Kód              Jelentés                       Mikor?
  ---------------- ------------------------------ ------------------------
  **400 Bad        Hibás formátumú adatok         Hiányzó vagy rossz mezők
  Request**                                       

  **401            Nincs vagy hibás token         Token hiányzik / hibás
  Unauthorized**                                  

  **403            Jogosultsági hiba              Nem admin → admin
  Forbidden**                                     végpont

  **404 Not        Nem található erőforrás        Felh., kurzus nem
  Found**                                         létezik

  **409 Conflict** Ütközés                        Már beiratkozott, már
                                                  completed

  **422            Validációs hiba                Pl. email már foglalt
  Unprocessable                                   
  Entity**                                        

  **503 Service    Szolgáltatás nem elérhető      Váratlan hiba esetén
  Unavailable**                                   
  ------------------------------------------------------------------------

------------------------------------------------------------------------

# Hitelesítés

Az autentikált végpontokhoz szükséges header:

    Authorization: Bearer <token>
    Accept: application/json

------------------------------------------------------------------------

# **Végpontok**

## 1. Nyilvános végpontok

### **GET /ping**

API működésének tesztelése.

**Válasz:**

``` json
{ "message": "API works!" }
```

### **POST /register**

Új felhasználó létrehozása.

**Body:**

``` json
{
  "name": "mozso",
  "email": "mozso@moriczref.hu",
  "password": "Jelszo_2025",
  "password_confirmation": "Jelszo_2025"
}
```

**201 Created**

``` json
{
  "message": "User created successfully",
  "user": {
    "id": 13,
    "name": "mozso",
    "email": "mozso@moriczref.hu",
    "role": "student"
  }
}
```

**Ha email foglalt -- 422:**

``` json
{
  "message": "Failed to register user",
  "errors": {
    "email": ["The email has already been taken."]
  }
}
```

### **POST /login**

**Body:**

``` json
{
  "email": "mozso@moriczref.hu",
  "password": "Jelszo_2025"
}
```

**Siker:**

``` json
{
  "message": "Login successful",
  "user": { },
  "access": {
    "token": "2|7F....",
    "token_type": "Bearer"
  }
}
```

**Hiba:**

``` json
{ "message": "Invalid email or password" }
```

------------------------------------------------------------------------

# **Autentikált végpontok**

## **POST /logout**

Kilépteti a felhasználót.

``` json
{ "message": "Logout successful" }
```

------------------------------------------------------------------------

# **Felhasználó kezelés**

## **GET /users/me**

``` json
{
  "user": {
    "id": 1,
    "name": "admin",
    "email": "admin@example.com",
    "role": "admin"
  },
  "stats": {
    "enrolledCourses": 10,
    "completedCourses": 11
  }
}
```

## **PUT /users/me**

Profil módosítása.

------------------------------------------------------------------------

# **Admin végpontok**

## **GET /users**

Admin listázza a felhasználókat.

## **GET /users/:id**

Admin lekér egy konkrét felhasználót.

## **DELETE /users/:id**

Soft delete felhasználó.

------------------------------------------------------------------------

# **Kurzusok**

## **GET /courses**

Kurzuslista.

## **GET /courses/:id**

Kurzus részletei + beiratkozók.

## **POST /courses/:id/enroll**

Beiratkozás kurzusra.

## **PATCH /courses/:id/completed**

Kurzus teljesítése.

------------------------------------------------------------------------

# Összefoglaló táblázat

  Metódus   Útvonal                  Jogosultság   Státuszok       Leírás
  --------- ------------------------ ------------- --------------- -----------------
  GET       /ping                    Public        200             API teszt
  POST      /register                Public        201, 422        Regisztráció
  POST      /login                   Public        200, 401        Login
  POST      /logout                  Auth          200             Logout
  GET       /users/me                Auth          200             Saját adatok
  PUT       /users/me                Auth          200             Saját módosítás
  GET       /users                   Admin         200             Lista
  GET       /users/:id               Admin         200, 404        Egy user
  DELETE    /users/:id               Admin         200, 404        Törlés
  GET       /courses                 Auth          200             Kurzuslista
  GET       /courses/:id             Auth          200, 404        Kurzus
  POST      /courses/:id/enroll      Auth          200, 409        Beiratkozás
  PATCH     /courses/:id/completed   Auth          200, 403, 409   Teljesítés
