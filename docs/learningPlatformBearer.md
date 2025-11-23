# Tanulási platform REST API megvalósítása Laravel környezetben

base_url: http://127.0.0.1/learningPlatformBearer/public/api vagy http://127.0.0.1:8000/api

Az API-t olyan funkciókkal kell ellátni, amelyek lehetővé teszik annak nyilvános elérhetőségét. Ennek a backendnek a fő célja, hogy kiszolgálja a frontendet, amelyet a tanulók a kurzusokra való feliratkozásra és a tanulásra használnak.

## Funkciók

Authentikáció (login, token kezelés).

Felhasználó beiratkozhat egy kurzusra.

Kurzuson belül a kurzus teljesítését jelöljük.

A teszteléshez készíts

- 1 admin (admin / admin)

- 9 student user (jelszó: Jelszo_2025)

- 3 releváns kurzus

- két véletlen student beiratkozásai (köztük 1-1 befejezett)

Az adatbázis neve: learning_platform.

## Végpontok

A Content-Type és az Accept headerkulcsok mindig application/json formátumúak legyenek.

Érvénytelen vagy hiányzó token esetén a backendnek 401 Unauthorized választ kell visszaadnia:

```php
JSON
```

Response: 401 Unauthorized

{

"message": "Invalid token"

}

Nem védett végpontok:

GET /ping - teszteléshez

POST /users/register - regisztrációhoz

POST /users/login - belépéshez

Hibák:

- 400 Bad Request: A kérés hibás formátumú. Ezt a hibát akkor kell visszaadni, ha a kérés hibásan van formázva, vagy ha hiányoznak a szükséges mezők.

- 401 Unauthorized: A felhasználó nem jogosult a kérés végrehajtására. Ezt a hibát akkor kell visszaadni, ha az érvénytelen a token.

- 403 Forbidden: A felhasználó nem jogosult a kérés végrehajtására. Ezt a hibát akkor kell visszaadni, ha a kreditkeret túllépésre kerül.

- 404 Not Found: A kért erőforrás nem található. Ezt a hibát akkor kell visszaadni, ha a kért kurzus, alkalom vagy foglalás nem található.

- 503 Service Unavailable: A szolgáltatás nem elérhető. Ezt a hibát akkor kell visszaadni, ha a tanulási szolgáltatás nem elérhető, vagy ha váratlan hibát ad vissza.

## Felhasználókezelés

- ------------------

POST /register

Új felhasználó regisztrálása. Az új felhasználók alapértelmezetten 0 kredittel rendelkeznek. Az e-mail címnek egyedinek kell lennie.

Kérés Törzse:

```php
JSON
```

{

"name": "mozso",

"email": "mozso@moriczref.hu",

"password" : "Jelszo_2025",

"password_confirmation" : "Jelszo_2025"

}

Válasz (sikeres regisztráció esetén): 201 Created

```php
JSON
```

{

"message": "User created successfully",

"user": {

"id": 13,

"name": "mozso",

"email": "mozso@moriczref.hu",

"role": "student"

}

}

Automatikus válasz felüldefiniálása (ha az e-mail cím már foglalt): 422 Unprocessable Entity

```php
JSON
```

{

"message": "Failed to register user",

"errors": {

"email": [

"The email has already been taken."

]

}

}

POST /login

Bejelentkezés e-mail címmel és jelszóval.

Kérés Törzse:

```php
JSON
```

{

"email": "mozso@moriczref.hu",

"password": "Jelszo_2025"

}

Válasz (sikeres bejelentkezés esetén): 200 OK

```php
JSON
```

{

"message": "Login successful",

"user": {

"id": 13,

"name": "mozso",

"email": "mozso@moriczref.hu",

"role": "student"

},

"access": {

"token": "2|7Fbr79b5zn8RxMfOqfdzZ31SnGWvgDidjahbdRfL2a98cfd8",

"token_type": "Bearer"

}

}

Válasz (sikertelen bejelentkezés esetén): 401 Unauthorized

```php
JSON
```

{

"message": "Invalid email or password"

}

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

! Az innen következő végpontok autentikáltak, tehát a kérés headerjében meg kell adni a tokent is !

! Authorization: "Bearer 2|7Fbr79b5zn8RxMfOqfdzZ31SnGWvgDidjahbdRfL2a98cfd8"                      !

!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!

POST /logout

A jelenlegi autentikált felhasználó kijelentkeztetése, a felhasználó tokenjének törlése. Ha a token érvénytelen, a fent meghatározott általános 401 Unauthorized hibát kell visszaadnia.

Válasz (sikeres kijelentkezés esetén): 200 OK

```php
JSON
```

{

"message": "Logout successful"

}

GET /users/me

Saját felhasználói profil, statisztikák lekérése.

Válasz: 200 OK

```php
JSON
```

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

PUT /users/me

Saját felhasználói adatok frissítése. Az aktuális felhasználó módosíthatja a nevét, e-mail címét és/vagy jelszavát.

Kérés törzse:

{

"name": "Új Név",

"email": "ujemail@example.com",

"password": "ÚjJelszo_2025",

"password_confirmation": "ÚjJelszo_2025"

}

Válasz (sikeres frissítés, 200 OK):

```php
JSON{
```

"message": "Profile updated successfully",

"user": {

"id": 5,

"name": "Új Név",

"email": "ujemail@example.com",

"role": "admin"

}

}

Hibák:

422 Unprocessable Entity – érvénytelen vagy hiányzó mezők, pl. nem egyezik a password_confirmation, vagy az e-mail már foglalt

401 Unauthorized – ha a token érvénytelen vagy hiányzik

GET /users

A felhasználói profilok, statisztikák lekérése az admin számára.

Válasz: 200 OK

```php
JSON
```

{

"data": [

{

"user": {

"id": 1,

"name": "admin",

"email": "admin@example.com",

"role": "admin"

},

"stats": {

"enrolledCourses": 10,

"completedCourses": 6

}

},

{

"user": {

"id": 2,

"name": "Aranka Török",

"email": "atorok@example.com",

"role": "student"

},

"stats": {

"enrolledCourses": 2,

"completedCourses": 1

}

},

{

"user": {

"id": 3,

"name": "Barnabás Török",

"email": "btorok@example.net",

"role": "student"

},

"stats": {

"enrolledCourses": 2,

"completedCourses": 1

}

}

]

}

Ha nem admin próbálja elérni a végpontot:

Válasz: 403 Forbidden

```php
JSON
```

{

"message": "Admin access required"

}

GET /users/:id

A felhasználói profil, statisztikák lekérése az admin számára.

Válasz: 200 OK

```php
JSON
```

{

"user": {

"id": 5,

"name": "Eva Rodriguez",

"email": "eva@example.com",

"role": "student",

},

"stats": {

"enrolledCourses": 3,

"completedCourses": 13

}

}

Ha nem admin próbálja elérni a végpontot:

Válasz: 403 Forbidden

```php
JSON
```

{

"message": "Admin access required"

}

Ha törölt (softdeleted) felhasználót próbáltunk megnézni:

Válasz: 404 Not Found

```php
JSON
```

{

"message": "User not found"

}

DELETE /users/:id

Egy felhasználó törlése (Soft Delete) az admin számára.

Ha a felhasználó már törlésre került, vagy nem létezik, a megfelelő hibaüzenetet adja vissza.

Válasz (sikeres törlés esetén): 200 OK

```php
JSON
```

{

"message": "User deleted successfully"

}

Válasz (ha a felhasználó nem található): 404 Not Found

```php
JSON
```

{

"message": "User not found"

}

Válasz (ha a token érvénytelen vagy hiányzik): 401 Unauthorized

```php
JSON
```

{

"message": "Invalid token"

}

## Kurzuskezelés

- -------------

GET /courses

Az összes elérhető kurzus listájának lekérése.

Válasz: 200 OK

```php
JSON
```

{

"courses": [

{

"title": "Szoftverfejlesztési alapok",

"description": "Alapvető programozási fogalmak és minták."

},

{

"title": "REST API fejlesztés",

"description": "API-k tervezése és készítése Laravelben."

}

]

}

GET /courses/:id

Információk lekérése egy adott kurzusról.

Válasz: 200 OK

```php
JSON
```

{

"course": {

"title": "Szoftverfejlesztési alapok",

"description": "Alapvető programozási fogalmak és minták."

},

"students": [

{

"name": "Barnabás Török",

"email": "btorok@example.net",

"completed": false,

},

{

"name": "Andrea Török",

"email": "atorok@example.net",

"completed": true,

}

]

}

Automatikus válasz (ha a kurzus nem található): 404 Not Found

POST /courses/:id/enroll

A jelenlegi felhasználó beiratkozása egy kurzusra.

Válasz (sikeres beiratkozás esetén): 200 OK

```php
JSON
```

{

"message": "Successfully enrolled in course"

}

Válasz (ha már beiratkozott): 409 Conflict

```php
JSON
```

{

"message": "Already enrolled in this course"

}

Automatikus válasz (ha a kurzus nem található): 404 Not Found

PATCH /courses/:id/completed

Jelenlegi felhasználó egy kurzusának befejezettként való megjelölése.

Válasz (sikeres befejezés esetén): 200 OK

```php
JSON
```

{

"message": "Course completed",

}

Válasz (ha nincs beiratkozva): 403 Forbidden

```php
JSON
```

{

"message": "Not enrolled in this course"

}

Válasz (ha már befejezett): 409 Conflict

```php
JSON
```

{

"message": "Course already completed"

}

## Összefoglalva

HTTP metódus	Útvonal	            Jogosultság	 Státuszkódok	                                        Rövid leírás

GET	        /ping	                Nyilvános	 200 OK	                                                API teszteléshez

POST	    /register	            Nyilvános	 201 Created, 400 Bad Request	                        Új felhasználó regisztrációja

POST	    /login	                Nyilvános	 200 OK, 401 Unauthorized	                            Bejelentkezés e-maillel és jelszóval

POST	    /logout	                Hitelesített 200 OK, 401 Unauthorized	                            Kijelentkezés

GET	        /users/me	            Hitelesített 200 OK, 401 Unauthorized	                            Saját profil és statisztikák lekérése

PUT         /users/me	            Hitelesített 200 OK, 422 Unprocessable Entity, 401 Unauthorized     Saját profil adatainak módosítása

GET	        /users  	            Admin	     200 OK, 403 Forbidden                               	Összes felhasználó profiljának lekérése

GET	        /users/:id	            Admin	     200 OK, 403 Forbidden, 404 Not Found, 401 Unauthorized	Bármely felhasználó profiljának lekérése

DELETE	    /users/:id	            Admin	     200 OK, 404 Not Found, 401 Unauthorized	            Felhasználó törlése (Soft Delete)

GET	        /courses	            Hitelesített 200 OK, 401 Unauthorized	                            Kurzusok listázása a beiratkozási státusszal

GET	        /courses/:id	        Hitelesített 200 OK, 404 Not Found, 401 Unauthorized	            Egy kurzus részletei

POST	    /courses/:id/enroll	    Hitelesített 200 OK, 409 Conflict, 404 Not Found, 401 Unauthorized	Beiratkozás kurzusra

PATCH	    /courses/:id/completed	Hitelesített 200 OK, 403 Forbidden, 409 Conflict, 401 Unauthorized	Kurzus befejezettként jelölése

## Adatbázis terv

+---------------------+     +---------------------+       +-----------------+        +------------+

|personal_access_tokens|    |        users        |       |   enrollments   |        |  courses   |

+---------------------+     +---------------------+       +-----------------+        +------------+

| id (PK)             |   _1| id (PK)             |1__    | id (PK)         |     __1| id (PK)    |

| tokenable_id (FK)   |K_/  | name                |   \__N| user_id (FK)    |    /   | title      |

| tokenable_type      |     | email (unique)      |       | course_id (FK)  |M__/    | description|

| name                |     | password            |       | enrolled_at     |        | created_at |

| token (unique)      |     | role (student/admin)|       | completed_at    |        | updated_at |

| abilities           |     | deleted_at          |       +-----------------+        +------------+

| last_used_at        |     +---------------------+

+---------------------+

*************************************

* I. Felvonás struktúra kialakítása *

*************************************

1. Telepítés (projekt létrehozása, .env konfiguráció, sanctum telepítése, tesztútvonal)

- --------------------------------------------------------------------------------------

célhely>composer create-project laravel/laravel --prefer-dist learningPlatformBearer

célhely>cd learningPlatformBearer

.env fájl módosítása

DB_CONNECTION=mysql

DB_HOST=127.0.0.1

DB_PORT=3306

DB_DATABASE=learning_platform

DB_USERNAME=root

DB_PASSWORD=

config/app.php módosítása

'timezone' => 'Europe/Budapest',

learningPlatformBearer>composer require laravel/sanctum

learningPlatformBearer>php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

learningPlatformBearer>php artisan install:api

api.php:

```php
use Illuminate\Support\Facades\Route;
```

```php
Route::get('/ping', function () {
```

```php
return response()->json([
```

'message' => 'API works!'

], 200);

});

learningPlatformBearer>php artisan serve

POSTMAN teszt: GET http://127.0.0.1/learningPlatform/public/api/ping

2. Modellek és migráció (sémák)

- -------------------------------

Ami már megvan (database/migrations) Ehhez nem is kell nyúlni

```php
Schema::create('personal_access_tokens', function (Blueprint $table) {
```

```php
$table->id();
```

```php
$table->morphs('tokenable'); // user kapcsolat
```

```php
$table->string('name');
```

```php
$table->string('token', 64)->unique();
```

```php
$table->text('abilities')->nullable();
```

```php
$table->timestamp('last_used_at')->nullable();
```

```php
$table->timestamps();
```

});

Ezt módosítani kell:

```php
Schema::create('users', function (Blueprint $table) {
```

```php
$table->id();
```

```php
$table->string('name');
```

```php
$table->string('email')->unique();
```

```php
$table->string('password');
```

//ezt bele kell írni

```php
$table->enum('role', ['student', 'admin']);
```

//ezt bele kell írni

```php
$table->softDeletes(); // ez adja hozzá a deleted_at mezőt
```

```php
$table->timestamps();
```

});

app/Models/User.php (módosítani kell)

```php
namespace App\Models;
```

```php
use Illuminate\Foundation\Auth\User as Authenticatable;
```

```php
use Laravel\Sanctum\HasApiTokens;
```

```php
use Illuminate\Notifications\Notifiable;
```

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;
```

```php
class User extends Authenticatable
```

{

```php
use HasApiTokens, HasFactory, Notifiable;
```

```php
protected $fillable = [
```

'name',

'email',

'password',

'role',

];

//amikor a modellt JSON formátumban adod vissza ne jelenjenek meg a következő mezők:

```php
protected $hidden = [
```

'password',

'remember_token',

];

```php
public function enrollments()
```

{

```php
return $this->hasMany(Enrollment::class);
```

}

```php
public function courses()
```

{

```php
return $this->belongsToMany(Course::class, 'enrollments');
```

}

```php
public function isAdmin()
```

{

```php
return $this->role === 'admin';
```

}

}

learningPlatformBearer>php artisan make:model Course -m

database/migrations/*_create_courses_table.php (módosítani kell)

```php
Schema::create('courses', function (Blueprint $table) {
```

```php
$table->id();
```

```php
$table->string('title');
```

```php
$table->text('description')->nullable();
```

```php
$table->timestamps();
```

});

app/Models/Course.php (módosítani kell)

```php
namespace App\Models;
```

```php
use Illuminate\Database\Eloquent\Model;
```

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;
```

```php
class Course extends Model
```

{

```php
use HasFactory;
```

```php
protected $fillable = [
```

'title',

'description',

];

```php
public function enrollments()
```

{

```php
return $this->hasMany(Enrollment::class);
```

}

```php
public function users()
```

{

```php
return $this->belongsToMany(User::class, 'enrollments');
```

}

}

learningPlatformBearer>php artisan make:model Enrollment -m

database/migrations/*_create_enrollments_table.php (módosítani kell)

```php
Schema::create('enrollments', function (Blueprint $table) {
```

```php
$table->id();
```

```php
$table->foreignId('user_id')->constrained()->cascadeOnDelete();
```

//a user_id mező a users tábla id oszlopára fog hivatkozni

```php
$table->foreignId('course_id')->constrained()->cascadeOnDelete();
```

```php
$table->timestamp('enrolled_at')->useCurrent();
```

```php
$table->timestamp('completed_at')->nullable(); // jelzi, hogy a kurzus befejeződött
```

});

app/Models/Enrollment.php (módosítani kell)

```php
namespace App\Models;
```

```php
use Illuminate\Database\Eloquent\Model;
```

```php
use Illuminate\Database\Eloquent\Factories\HasFactory;
```

```php
class Enrollment extends Model
```

{

```php
use HasFactory;
```

```php
public $timestamps = false;
```

```php
protected $fillable = [
```

'user_id',

'course_id',

'enrolled_at',

'completed_at',

];

```php
protected $dates = [
```

'enrolled_at',

'completed_at',

];

```php
public function user()
```

{

```php
return $this->belongsTo(User::class);
```

}

```php
public function course()
```

{

```php
return $this->belongsTo(Course::class);
```

}

}

learningPlatformBearer>php artisan migrate

3. Seeding (Factory és seederek)

- -------------------------------

database/factories/UserFactory.php (módosítása)

```php
namespace Database\Factories;
```

```php
use Illuminate\Database\Eloquent\Factories\Factory;
```

```php
use Illuminate\Support\Facades\Hash;
```

```php
use App\Models\User;
```

```php
class UserFactory extends Factory
```

{

```php
protected $model = User::class;
```

```php
public function definition()
```

{

```php
$this->faker = \Faker\Factory::create('hu_HU'); // magyar nevekhez
```

```php
return [
```

'name' => $this->faker->firstName . ' ' . $this->faker->lastName, // magyaros teljes név

'email' => $this->faker->unique()->safeEmail(),

'password' => Hash::make('Jelszo123'), // minden user jelszava: Jelszo123

'role' => 'student',

];

}

}

learningPlatformBearer>php artisan make:seeder UserSeeder

database/seeders/UserSeeder.php (módosítása)

```php
namespace Database\Seeders;
```

```php
use Illuminate\Database\Seeder;
```

```php
use App\Models\User;
```

```php
use Illuminate\Support\Facades\Hash;
```

```php
class UserSeeder extends Seeder
```

{

```php
public function run(): void
```

{

// 1 admin

User::create([

'name' => 'admin',

'email' => 'admin@example.com',

'password' => Hash::make('admin'),

'role' => 'admin',

]);

// 9 student user

User::factory(9)->create();

}

}

learningPlatformBearer>php artisan make:seeder CourseSeeder

database/seeders/CourseSeeder.php (módosítása)

```php
namespace Database\Seeders;
```

```php
use Illuminate\Database\Seeder;
```

```php
use App\Models\Course;
```

```php
class CourseSeeder extends Seeder
```

{

```php
public function run(): void
```

{

Course::create([

'title' => 'Szoftverfejlesztési alapok',

'description' => 'Alapvető programozási fogalmak és minták.',

]);

Course::create([

'title' => 'REST API fejlesztés',

'description' => 'API-k tervezése és készítése Laravelben.',

]);

Course::create([

'title' => 'Fullstack webfejlesztés',

'description' => 'Backend és frontend alapok.',

]);

}

}

learningPlatformBearer>php artisan make:seeder EnrollmentSeeder

database/seeders/EnrollmentSeeder.php (módosítása)

```php
namespace Database\Seeders;
```

```php
use Illuminate\Database\Seeder;
```

```php
use App\Models\User;
```

```php
use App\Models\Course;
```

```php
use App\Models\Enrollment;
```

```php
use Carbon\Carbon;
```

```php
class EnrollmentSeeder extends Seeder
```

{

```php
public function run(): void
```

{

```php
$students = User::where('role', 'student')->take(2)->get();
```

```php
$courses = Course::all();
```

// User 1: első két kurzus

Enrollment::create([

'user_id' => $students[0]->id,

'course_id' => $courses[0]->id,

'enrolled_at' => now(),

'completed_at' => now(),  // completed

]);

Enrollment::create([

'user_id' => $students[0]->id,

'course_id' => $courses[1]->id,

'enrolled_at' => now(),

'completed_at' => null,

]);

// User 2: első két kurzus

Enrollment::create([

'user_id' => $students[1]->id,

'course_id' => $courses[0]->id,

'enrolled_at' => now(),

'completed_at' => now(), // completed

]);

Enrollment::create([

'user_id' => $students[1]->id,

'course_id' => $courses[2]->id,

'enrolled_at' => now(),

'completed_at' => null,

]);

}

}

DatabaseSeeder.php (módosítása)

```php
namespace Database\Seeders;
```

```php
use Illuminate\Database\Seeder;
```

```php
class DatabaseSeeder extends Seeder
```

{

```php
public function run(): void
```

{

```php
$this->call([
```

UserSeeder::class,

CourseSeeder::class,

EnrollmentSeeder::class,

]);

}

}

learningPlatformBearer>php artisan db:seed

******************************************

* II. Felvonás Controller és endpoint-ok *

******************************************

learningPlatformBearer>php artisan make:controller AuthController

app\Http\Controllers\AuthController.php szerkesztése

```php
namespace App\Http\Controllers;
```

```php
use App\Models\User;
```

```php
use Illuminate\Http\Request;
```

```php
use Illuminate\Support\Facades\Hash;
```

```php
use Illuminate\Validation\ValidationException;
```

```php
class AuthController extends Controller
```

{

```php
public function register(Request $request)
```

{

try {

```php
$request->validate([
```

'name' => 'required|string|max:255',

'email' => 'required|email|unique:users,email',

'password' => 'required|string|confirmed|min:8',

]);

} catch (ValidationException $e) {

```php
return response()->json([
```

'message' => 'Failed to register user',

'errors' => $e->errors() // visszaadja, mely mezők hibásak

], 422);

}

```php
$user = User::create([
```

'name' => $request->name,

'email' => $request->email,

'password' => Hash::make($request->password),

'role' => 'student',

]);

```php
return response()->json([
```

'message' => 'User created successfully',

'user' => [

'id' => $user->id,

'name' => $user->name,

'email' => $user->email,

'role' => $user->role,

],

], 201);

}

```php
public function login(Request $request)
```

{

```php
$user = User::where('email', $request->email)->first();
```

if (!$user || !Hash::check($request->password, $user->password)) {

```php
return response()->json(['message' => 'Invalid email or password'], 401);
```

}

```php
$token = $user->createToken('api-token')->plainTextToken;
```

```php
return response()->json([
```

'message' => 'Login successful',

'user' => [

'id' => $user->id,

'name' => $user->name,

'email' => $user->email,

'role' => $user->role,

],

'access' => [

'token' => $token,

'token_type' => 'Bearer'

]

]);

}

```php
public function logout(Request $request)
```

{

```php
$request->user()->tokens()->delete(); //minden token törlése
```

//$request->user()->currentAccessToken()->delete(); //aktuális token törlése, más eszközökön marad a bejelentkezés

```php
return response()->json(['message' => 'Logout successful']);
```

}

}

learningPlatformBearer>php artisan make:controller UserController

app\Http\Controllers\UserController.php szerkesztése

```php
namespace App\Http\Controllers;
```

```php
use App\Models\User;
```

```php
use Illuminate\Http\Request;
```

```php
class UserController extends Controller
```

{

/**

* GET /users/me

* A bejelentkezett felhasználó adatainak lekérése.

*/

```php
public function me(Request $request)
```

{

```php
$user = $request->user();
```

```php
return response()->json([
```

'user' => [

'id'    => $user->id,

'name'  => $user->name,

'email' => $user->email,

'role'  => $user->role,

],

'stats' => [

'enrolledCourses'  => $user->enrollments()->count(),

'completedCourses' => $user->enrollments()->where('completed_at', true)->count(),

]

], 200);

}

/**

* PUT /users/me

* A bejelentkezett felhasználó adatainak frissítése.

*/

```php
public function updateMe(Request $request)
```

{

```php
$user = $request->user();
```

```php
$request->validate([
```

'name'   => 'sometimes|string|max:255',

'email'  => 'sometimes|email|unique:users,email,' . $user->id,

'password' => 'sometimes|string|confirmed|min:8',

]);

if ($request->name) {

```php
$user->name = $request->name;
```

}

if ($request->email) {

```php
$user->email = $request->email;
```

}

if ($request->password) {

```php
$user->password = bcrypt($request->password);
```

}

```php
$user->save();
```

```php
return response()->json([
```

'message' => 'Profile updated successfully',

'user' => [

'id'    => $user->id,

'name'  => $user->name,

'email' => $user->email,

'role'  => $user->role,

],

]);

}

/**

* ADMIN ONLY

* GET /users

* Összes felhasználó listázása.

*/

```php
public function index(Request $request)
```

{

if ($request->user()->role !== 'admin') {

```php
return response()->json(['message' => 'Forbidden'], 403);
```

}

```php
$users = User::all()->map(function ($user) {
```

```php
return [
```

'user' => [

'id' => $user->id,

'name' => $user->name,

'email' => $user->email,

'role' => $user->role,

],

'stats' => [

'enrolledCourses'  => $user->enrollments()->count(),

'completedCourses' => $user->enrollments()->whereNotNull('completed_at')->count(),

]

];

});

```php
return response()->json([
```

'data' => $users

]);

}

/**

* ADMIN ONLY

* GET /users/{id}

* Felhasználó lekérése ID alapján.

*/

```php
public function show(Request $request, $id)
```

{

if ($request->user()->role !== 'admin') {

```php
return response()->json(['message' => 'Forbidden'], 403);
```

}

```php
$user = User::withTrashed()->find($id);
```

if (!$user) {

```php
return response()->json(['message' => 'User not found'], 404);
```

}

if ($user->trashed()) {

```php
return response()->json(['message' => 'User is deleted'], 404);
```

}

```php
return response()->json([
```

'user' => [

'id'    => $user->id,

'name'  => $user->name,

'email' => $user->email,

'role'  => $user->role,

],

'stats' => [

'enrolledCourses'  => $user->enrollments()->count(),

'completedCourses' => $user->enrollments()->whereNotNull('completed_at')->count(),

]

]);

}

/**

* ADMIN ONLY

* DELETE /users/{id}

* Soft delete felhasználó.

*/

```php
public function destroy(Request $request, $id)
```

{

if ($request->user()->role !== 'admin') {

```php
return response()->json(['message' => 'Forbidden'], 403);
```

}

```php
$user = User::find($id);
```

if (!$user) {

```php
return response()->json(['message' => 'User not found'], 404);
```

}

```php
$user->delete();
```

```php
return response()->json(['message' => 'User deleted successfully']);
```

}

}

learningPlatformBearer>php artisan make:controller CourseController

app\Http\Controllers\CourseController.php szerkesztése

```php
namespace App\Http\Controllers;
```

```php
use App\Models\Course;
```

```php
use App\Models\Enrollment;
```

```php
use Illuminate\Http\Request;
```

```php
class CourseController extends Controller
```

{

```php
public function index(Request $request)
```

{

```php
$courses = Course::select('title', 'description')->get();
```

```php
return response()->json([
```

'courses' => $courses

]);

}

```php
public function show(Course $course)
```

{

// Csak a szükséges mezők a kapcsolt usereknél, valamint a teljesítési státusz

```php
$students = $course->users()->select('name', 'email')->withPivot('completed_at')->get()->map(function ($user) {
```

```php
return [
```

'name' => $user->name,

'email' => $user->email,

'completed' => !is_null($user->pivot->completed_at)

];

});

```php
return response()->json([
```

'course' => [

'title' => $course->title,

'description' => $course->description

],

'students' => $students

]);

}

```php
public function enroll(Course $course, Request $request)
```

{

```php
$user = $request->user();
```

if ($user->courses()->where('course_id', $course->id)->exists()) {

```php
return response()->json(['message' => 'Already enrolled in this course'], 409);
```

}

```php
$user->courses()->attach($course->id, ['enrolled_at' => now()]);
```

```php
return response()->json(['message' => 'Successfully enrolled in course']);
```

}

```php
public function complete(Course $course, Request $request)
```

{

```php
$user = $request->user();
```

```php
$enrollment = Enrollment::where('user_id', $user->id)
```

- >where('course_id', $course->id)

- >first();

if (! $enrollment) {

```php
return response()->json(['message' => 'Not enrolled in this course'], 403);
```

}

if ($enrollment->completed_at) {

```php
return response()->json(['message' => 'Course already completed'], 409);
```

}

```php
$enrollment->update(['completed_at' => now()]);
```

```php
return response()->json(['message' => 'Course completed']);
```

}

}

routes\api.php frissítése:

```php
use Illuminate\Support\Facades\Route;
```

```php
use App\Http\Controllers\AuthController;
```

```php
use App\Http\Controllers\UserController;
```

```php
use App\Http\Controllers\CourseController;
```

// Public

```php
Route::get('/ping', function () { return response()->json(['message'=>'API works!']); });
```

```php
Route::post('/register', [AuthController::class, 'register']);
```

```php
Route::post('/login', [AuthController::class, 'login']);
```

// Authenticated

```php
Route::middleware('auth:sanctum')->group(function () {
```

```php
Route::post('/logout', [AuthController::class, 'logout']);
```

```php
Route::get('/users/me', [UserController::class, 'me']);
```

```php
Route::put('/users/me', [UserController::class, 'updateMe']);
```

// Admin

```php
Route::get('/users', [UserController::class, 'index']);
```

```php
Route::get('/users/{id}', [UserController::class, 'show']);
```

```php
Route::delete('/users/{id}', [UserController::class, 'destroy']);
```

```php
Route::get('/courses', [CourseController::class, 'index']);
```

```php
Route::get('/courses/{course}', [CourseController::class, 'show']);
```

```php
Route::post('/courses/{course}/enroll', [CourseController::class, 'enroll']);
```

```php
Route::patch('/courses/{course}/completed', [CourseController::class, 'complete']);
```

});

***************************

* III. felvonás Tesztelés *

***************************

Feature teszt ideális az HTTP kérések szimulálására, mert több komponens (Controller, Middleware, Auth) együttműködését vizsgáljuk.

learningPlatformBearer>php artisan make:test AuthTest

```php
namespace Tests\Feature;
```

```php
use Illuminate\Foundation\Testing\RefreshDatabase;
```

```php
use Illuminate\Foundation\Testing\WithFaker;
```

```php
use Tests\TestCase;
```

```php
use App\Models\User;
```

```php
use Illuminate\Support\Facades\Hash;
```

```php
class AuthTest extends TestCase
```

{

```php
use RefreshDatabase;
```

```php
public function test_ping_endpoint_returns_ok()
```

{

```php
$response = $this->getJson('/api/ping');
```

```php
$response->assertStatus(200)
```

- >assertJson(['message' => 'API works!']);

}

```php
public function test_register_creates_user()
```

{

```php
$payload = [
```

'name' => 'Teszt Elek',

'email' => 'teszt@example.com',

'password' => 'Jelszo_2025',

'password_confirmation' => 'Jelszo_2025'

];

```php
$response = $this->postJson('/api/register', $payload);
```

```php
$response->assertStatus(201)
```

- >assertJsonStructure(['message', 'user' => ['id', 'name', 'email', 'role']]);

// Ellenőrizzük, hogy a felhasználó létrejött az adatbázisban

```php
$this->assertDatabaseHas('users', [
```

'email' => 'teszt@example.com',

]);

}

```php
public function test_login_with_valid_credentials()
```

{

// ARRANGE: Felhasználó létrehozása az adatbázisban

// Mivel a regisztrációs teszt csak egyszer fut, létre kell hozni egy felhasználót

// minden login teszthez.

```php
$password = 'Jelszo_2025';
```

```php
$user = User::factory()->create([
```

'email' => 'validuser@example.com',

'password' => Hash::make($password), // A jelszót hash-elni kell!

]);

// ACT: Bejelentkezési kérés

```php
$response = $this->postJson('/api/login', [
```

'email' => 'validuser@example.com',

'password' => $password, // A bejelentkezéshez a plain text jelszót adjuk

]);

// ASSERT: Ellenőrizzük a státuszt és a válasz struktúráját

```php
$response->assertStatus(200)
```

- >assertJsonStructure(['message', 'user' => ['id', 'name', 'email', 'role'], 'access' => ['token', 'token_type']]);

// Opcionális: Ellenőrizzük, hogy létrejött-e token

```php
$this->assertDatabaseHas('personal_access_tokens', [
```

'tokenable_id' => $user->id,

]);

}

```php
public function test_login_with_invalid_credentials()
```

{

// ARRANGE: Létrehozzuk a létező felhasználót

```php
$user = User::factory()->create([
```

'email' => 'existing@example.com',

'password' => Hash::make('CorrectPassword'),

]);

// ACT: Helytelen adatokkal próbálkozunk

```php
$response = $this->postJson('/api/login', [
```

'email' => 'existing@example.com',

'password' => 'wrongpass' // Helytelen jelszó

]);

// ASSERT: Ellenőrizzük az elutasítást

// FIGYELEM: Ha a backend 422-t ad vissza validációs hiba (pl. hiányzó mező) helyett,

// de az invalid credentials hiba 401, akkor az a helyes.

```php
$response->assertStatus(401)
```

- >assertJson(['message' => 'Invalid email or password']);

}

}

learningPlatformBearer>php artisan make:test UserTest

```php
namespace Tests\Feature;
```

```php
use App\Models\User;
```

```php
use Illuminate\Foundation\Testing\RefreshDatabase;
```

```php
use Tests\TestCase;
```

```php
use Laravel\Sanctum\Sanctum;
```

```php
use Illuminate\Support\Facades\Hash;
```

```php
use Illuminate\Testing\Fluent\AssertableJson;
```

```php
class UserTest extends TestCase
```

{

```php
use RefreshDatabase; // Az adatbázis hibák (no such table: users) elkerülésére
```

// ----------------------------------------------------------------------------------

// 1. /users/me (GET) - Lekérés

// ----------------------------------------------------------------------------------

```php
public function test_me_endpoint_requires_authentication()
```

{

// A hiba alapján a Laravel alapértelmezett üzenetét várjuk

```php
$response = $this->getJson('/api/users/me');
```

```php
$response->assertStatus(401)
```

- >assertJson(['message' => 'Unauthenticated.']);

}

```php
public function test_me_endpoint_returns_user_data()
```

{

// ARRANGE: Felhasználó létrehozása

```php
$user = User::factory()->create(['role' => 'student']);
```

// ACT: Felhasználó hitelesítése a Sanctum-mal

Sanctum::actingAs($user);

// ACT: Kérés küldése

```php
$response = $this->getJson('/api/users/me');
```

// ASSERT: Ellenőrizzük a státuszt és a válasz struktúráját (userController.php alapján)

```php
$response->assertStatus(200)
```

- >assertJsonStructure([

'user' => ['id', 'name', 'email', 'role'],

'stats' => ['enrolledCourses', 'completedCourses']

])

// Ellenőrizzük, hogy a válasz a helyes felhasználót tartalmazza

- >assertJsonPath('user.email', $user->email);

}

// ----------------------------------------------------------------------------------

// 2. /users/me (PUT) - Profil Frissítés

// ----------------------------------------------------------------------------------

```php
public function test_user_can_update_their_own_name_and_email()
```

{

```php
$user = User::factory()->create(['name' => 'Old Name', 'email' => 'old@example.com']);
```

Sanctum::actingAs($user);

```php
$newEmail = 'new@example.com';
```

```php
$newName = 'New Name';
```

```php
$response = $this->putJson('/api/users/me', [
```

'name' => $newName,

'email' => $newEmail,

]);

```php
$response->assertStatus(200)
```

- >assertJson(['message' => 'Profile updated successfully'])

- >assertJsonPath('user.name', $newName)

- >assertJsonPath('user.email', $newEmail);

// Ellenőrizzük az adatbázist

```php
$this->assertDatabaseHas('users', [
```

'id' => $user->id,

'name' => $newName,

'email' => $newEmail,

]);

}

```php
public function test_user_can_update_their_password()
```

{

```php
$user = User::factory()->create();
```

Sanctum::actingAs($user);

```php
$newPassword = 'New_Secure_Password_2025';
```

```php
$response = $this->putJson('/api/users/me', [
```

'password' => $newPassword,

'password_confirmation' => $newPassword,

]);

```php
$response->assertStatus(200);
```

// Frissítsük a felhasználót az adatbázisból és ellenőrizzük a jelszót

```php
$updatedUser = User::find($user->id);
```

```php
$this->assertTrue(Hash::check($newPassword, $updatedUser->password));
```

}

// ----------------------------------------------------------------------------------

// 3. /users (GET) - Összes felhasználó listázása (Admin Only)

// ----------------------------------------------------------------------------------

```php
public function test_student_cannot_access_user_list()
```

{

```php
$student = User::factory()->create(['role' => 'student']);
```

Sanctum::actingAs($student);

```php
$response = $this->getJson('/api/users');
```

// userController.php 120. sor: return response()->json(['message' => 'Forbidden'], 403);

```php
$response->assertStatus(403)
```

- >assertJson(['message' => 'Forbidden']);

}

```php
public function test_admin_can_access_user_list()
```

{

// ARRANGE: Létrehozunk egy admin és néhány student felhasználót

```php
$admin = User::factory()->create(['role' => 'admin']);
```

```php
$students = User::factory(3)->create(['role' => 'student']);
```

Sanctum::actingAs($admin);

```php
$response = $this->getJson('/api/users');
```

```php
$response->assertStatus(200)
```

- >assertJsonStructure(['data' => [

'*' => [

'user' => ['id', 'name', 'email', 'role'],

'stats' => ['enrolledCourses', 'completedCourses']

]

]])

// Ellenőrizzük, hogy az összes felhasználót (admin + 3 student) visszaadta

- >assertJson(fn (AssertableJson $json) =>

```php
$json->has('data', 4)
```

- >etc()

);

}

// ----------------------------------------------------------------------------------

// 4. /users/{id} (GET) - Felhasználó Megtekintése (Admin Only)

// ----------------------------------------------------------------------------------

```php
public function test_admin_can_view_specific_user()
```

{

```php
$admin = User::factory()->create(['role' => 'admin']);
```

```php
$targetUser = User::factory()->create(['role' => 'student', 'name' => 'Target User']);
```

Sanctum::actingAs($admin);

```php
$response = $this->getJson("/api/users/{$targetUser->id}");
```

```php
$response->assertStatus(200)
```

- >assertJsonPath('user.name', 'Target User');

}

```php
public function test_student_cannot_view_other_users()
```

{

```php
$student = User::factory()->create(['role' => 'student']);
```

```php
$otherUser = User::factory()->create(['role' => 'student']);
```

Sanctum::actingAs($student);

```php
$response = $this->getJson("/api/users/{$otherUser->id}");
```

```php
$response->assertStatus(403)
```

- >assertJson(['message' => 'Forbidden']);

}

// ----------------------------------------------------------------------------------

// 5. /users/{id} (DELETE) - Felhasználó Törlése (Admin Only - Soft Delete)

// ----------------------------------------------------------------------------------

```php
public function test_admin_can_soft_delete_a_user()
```

{

```php
$admin = User::factory()->create(['role' => 'admin']);
```

```php
$userToDelete = User::factory()->create();
```

Sanctum::actingAs($admin);

```php
$response = $this->deleteJson("/api/users/{$userToDelete->id}");
```

```php
$response->assertStatus(200)
```

- >assertJson(['message' => 'User deleted successfully']);

// Ellenőrizzük, hogy a felhasználó soft deleted

```php
$this->assertSoftDeleted('users', ['id' => $userToDelete->id]);
```

}

```php
public function test_student_cannot_delete_users()
```

{

```php
$student = User::factory()->create(['role' => 'student']);
```

```php
$userToDelete = User::factory()->create();
```

Sanctum::actingAs($student);

```php
$response = $this->deleteJson("/api/users/{$userToDelete->id}");
```

```php
$response->assertStatus(403)
```

- >assertJson(['message' => 'Forbidden']);

// Ellenőrizzük, hogy a felhasználó NEM lett törölve

```php
$this->assertDatabaseHas('users', ['id' => $userToDelete->id]);
```

}

}

learningPlatformBearer>php artisan make:test CourseTest

```php
namespace Tests\Feature;
```

```php
use App\Models\Course;
```

```php
use App\Models\User;
```

```php
use Illuminate\Foundation\Testing\RefreshDatabase;
```

```php
use Tests\TestCase;
```

```php
use Laravel\Sanctum\Sanctum;
```

```php
use Illuminate\Testing\Fluent\AssertableJson;
```

```php
class CourseTest extends TestCase
```

{

```php
use RefreshDatabase; // Elengedhetetlen az adatbázis táblák létrehozásához
```

// ----------------------------------------------------------------------------------

// 1. /courses (GET) - Lista lekérése

// ----------------------------------------------------------------------------------

```php
public function test_course_index_requires_authentication()
```

{

```php
$response = $this->getJson('/api/courses');
```

```php
$response->assertStatus(401)
```

- >assertJson(['message' => 'Unauthenticated.']);

}

```php
public function test_course_index_returns_list_of_courses()
```

{

// ARRANGE: Felhasználó és 3 kurzus manuális létrehozása

```php
$user = User::factory()->create();
```

// <<< MANUÁLIS LÉTREHOZÁS FACTORY HELYETT

Course::create(['title' => 'Kurzus A', 'description' => 'Leírás A']);

Course::create(['title' => 'Kurzus B', 'description' => 'Leírás B']);

Course::create(['title' => 'Kurzus C', 'description' => 'Leírás C']);

Sanctum::actingAs($user);

// ACT: Kérés küldése

```php
$response = $this->getJson('/api/courses');
```

// ASSERT: Ellenőrizzük a státuszt és a struktúrát

```php
$response->assertStatus(200)
```

- >assertJsonStructure(['courses' => [

'*' => ['title', 'description']

]])

- >assertJson(fn (AssertableJson $json) =>

```php
$json->has('courses', 3) // Ellenőrizzük, hogy mindhárom kurzus visszajött
```

- >etc()

);

}

// ----------------------------------------------------------------------------------

// 2. /courses/{id} (GET) - Kurzus részletek

// ----------------------------------------------------------------------------------

```php
public function test_course_show_returns_details_and_students()
```

{

// ARRANGE: Admin és kurzus manuális létrehozása

```php
$user = User::factory()->create(['role' => 'admin']);
```

// <<< MANUÁLIS LÉTREHOZÁS FACTORY HELYETT

```php
$course = Course::create(['title' => 'Részletes Kurzus', 'description' => 'Részletes Leírás']);
```

```php
$student1 = User::factory()->create();
```

```php
$student2 = User::factory()->create();
```

// Beiratkozás: 1 beiratkozott, 1 befejezett

```php
$student1->courses()->attach($course->id, ['enrolled_at' => now()]);
```

```php
$student2->courses()->attach($course->id, ['enrolled_at' => now(), 'completed_at' => now()]);
```

Sanctum::actingAs($user);

// ACT: Kérés küldése

```php
$response = $this->getJson("/api/courses/{$course->id}");
```

// ASSERT: Ellenőrizzük a státuszt és a fészkelt struktúrát

```php
$response->assertStatus(200)
```

- >assertJsonPath('course.title', $course->title)

- >assertJson(fn (AssertableJson $json) =>

```php
$json->has('students', 2) // Két diák van
```

- >where('students.0.completed', false) // student1: nincs completed_at

- >where('students.1.completed', true) // student2: van completed_at

- >etc()

);

}

// ----------------------------------------------------------------------------------

// 3. /courses/{id}/enroll (POST) - Beiratkozás

// ----------------------------------------------------------------------------------

```php
public function test_user_can_enroll_in_a_course()
```

{

```php
$user = User::factory()->create();
```

// <<< MANUÁLIS LÉTREHOZÁS FACTORY HELYETT

```php
$course = Course::create(['title' => 'Beiratkozó Kurzus', 'description' => 'Leírás']);
```

Sanctum::actingAs($user);

// ACT: Beiratkozás

```php
$response = $this->postJson("/api/courses/{$course->id}/enroll");
```

```php
$response->assertStatus(200)
```

- >assertJson(['message' => 'Successfully enrolled in course']);

// ASSERT: Ellenőrizzük a kapcsolótáblát

```php
$this->assertDatabaseHas('enrollments', [
```

'user_id' => $user->id,

'course_id' => $course->id,

'completed_at' => null,

]);

}

```php
public function test_enrollment_fails_if_already_enrolled()
```

{

```php
$user = User::factory()->create();
```

// <<< MANUÁLIS LÉTREHOZÁS FACTORY HELYETT

```php
$course = Course::create(['title' => 'Már Beiratkozott Kurzus', 'description' => 'Leírás']);
```

Sanctum::actingAs($user);

// Először beiratkozunk

```php
$user->courses()->attach($course->id, ['enrolled_at' => now()]);
```

// ACT: Újra megpróbáljuk

```php
$response = $this->postJson("/api/courses/{$course->id}/enroll");
```

```php
$response->assertStatus(409)
```

- >assertJson(['message' => 'Already enrolled in this course']);

}

// ----------------------------------------------------------------------------------

// 4. /courses/{id}/completed (PATCH) - Teljesítés

// ----------------------------------------------------------------------------------

```php
public function test_user_can_complete_an_enrolled_course()
```

{

```php
$user = User::factory()->create();
```

// <<< MANUÁLIS LÉTREHOZÁS FACTORY HELYETT

```php
$course = Course::create(['title' => 'Teljesíthető Kurzus', 'description' => 'Leírás']);
```

Sanctum::actingAs($user);

// Beiratkozás

```php
$user->courses()->attach($course->id, ['enrolled_at' => now()]);
```

// ACT: Teljesítés

```php
$response = $this->patchJson("/api/courses/{$course->id}/completed");
```

```php
$response->assertStatus(200)
```

- >assertJson(['message' => 'Course completed']);

// ASSERT: Ellenőrizzük, hogy a completed_at be lett állítva (nem null)

```php
$this->assertDatabaseMissing('enrollments', [
```

'user_id' => $user->id,

'course_id' => $course->id,

'completed_at' => null,

]);

}

```php
public function test_complete_fails_if_not_enrolled()
```

{

```php
$user = User::factory()->create();
```

// <<< MANUÁLIS LÉTREHOZÁS FACTORY HELYETT

```php
$course = Course::create(['title' => 'Nem Beiratkozott Kurzus', 'description' => 'Leírás']);
```

Sanctum::actingAs($user);

// ACT: Teljesítés beiratkozás nélkül

```php
$response = $this->patchJson("/api/courses/{$course->id}/completed");
```

```php
$response->assertStatus(403)
```

- >assertJson(['message' => 'Not enrolled in this course']);

}

```php
public function test_complete_fails_if_already_completed()
```

{

```php
$user = User::factory()->create();
```

// <<< MANUÁLIS LÉTREHOZÁS FACTORY HELYETT

```php
$course = Course::create(['title' => 'Már Teljesített Kurzus', 'description' => 'Leírás']);
```

Sanctum::actingAs($user);

// Beiratkozás és teljesítés

```php
$user->courses()->attach($course->id, ['enrolled_at' => now(), 'completed_at' => now()]);
```

// ACT: Újra megpróbáljuk a teljesítést

```php
$response = $this->patchJson("/api/courses/{$course->id}/completed");
```

```php
$response->assertStatus(409)
```

- >assertJson(['message' => 'Course already completed']);

}

}

learningPlatformBearer>php artisan test