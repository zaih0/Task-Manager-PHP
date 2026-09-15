# Task-Manager-PHP
A simple full-stack Task Manager application built to learn PHP, Laravel, Angular, and Ionic together. The project demonstrates a clean REST API architecture where a single Laravel backend serves both a web frontend (Angular) and a mobile app (Ionic).

📋 Table of Contents
Project Overview

Architecture

Tech Stack

Features

Laravel Architecture Concepts

Request Lifecycle

Getting Started

Prerequisites

Backend Setup (Laravel)

Web Frontend Setup (Angular)

Mobile App Setup (Ionic)

API Endpoints

Project Structure

Learning Roadmap

Milestone Checklist

Tips for Learning

License

🎯 Project Overview
A personal task manager where users can:

Register and log in

Create, read, update, and delete (CRUD) tasks

Mark tasks as complete

Filter tasks by status

Why this project?

Covers the core Laravel concepts (routing, MVC, Eloquent, auth, APIs)

Reuses the same backend for both a web frontend (Angular) and a mobile app (Ionic)

Small enough to finish, deep enough to teach real patterns

🏗️ Architecture
text
┌─────────────────┐     ┌─────────────────┐
│  Angular (Web)  │     │  Ionic (Mobile) │
└────────┬────────┘     └────────┬────────┘
         │  HTTP/JSON (REST API)  │
         └───────────┬────────────┘
                     ▼
         ┌───────────────────────┐
         │   Laravel Backend     │
         │  (API + Logic + Auth) │
         └───────────┬───────────┘
                     ▼
              ┌─────────────┐
              │   MySQL     │
              └─────────────┘
Key idea: Laravel is the API and business logic layer. Angular and Ionic are clients that consume it. Build the backend once, use it everywhere.

🛠 Tech Stack
Layer	Technology	Purpose
Backend	PHP 8.2+ / Laravel 11	REST API, business logic, authentication
Auth	Laravel Sanctum	Token-based API authentication
Database	MySQL / SQLite	Data persistence
Web Frontend	Angular 17+	Browser-based UI
Mobile Frontend	Ionic 7 + Capacitor	iOS / Android app
HTTP Client	Angular HttpClient	Communicating with the API
✨ Features
✅ User registration and login (Sanctum token auth)

✅ Per-user task isolation (users only see their own tasks)

✅ Full CRUD on tasks

✅ Mark tasks as complete / incomplete

✅ Filter tasks by status

✅ Responsive web UI (Angular)

✅ Native mobile UI (Ionic)

🧩 Laravel Architecture Concepts
Laravel is MVC-based. Here's how the pieces fit together:

Concept	What It Does	Example in Task App
Routes	Maps URLs to code	POST /api/tasks → create task
Controllers	Handle requests, return responses	TaskController@store
Models	Represent DB tables (Eloquent ORM)	Task model
Migrations	Version control for your DB schema	create_tasks_table
Middleware	Filter requests (auth, CORS, etc.)	auth:sanctum
Requests	Validate incoming data	StoreTaskRequest
Resources	Format JSON output	TaskResource
Service Container / Providers	Dependency injection	Auto-injected into controllers
Policies	Authorize actions on resources	TaskPolicy@update
🔄 Request Lifecycle
Memorize this — it's the backbone of Laravel:

Request hits public/index.php

Kernel boots → loads Service Providers

Request passes through Middleware

Router matches the URL

Controller runs, uses Model for data

Resource shapes the response → JSON back to client

🚀 Getting Started
Prerequisites
Make sure you have these installed:

PHP 8.2+ — php -v

Composer — composer -V

Node.js 20+ and npm — node -v

MySQL or SQLite

Angular CLI — npm install -g @angular/cli

Ionic CLI — npm install -g @ionic/cli

Backend Setup (Laravel)
bash
# Create the project
composer create-project laravel/laravel task-api
cd task-api

# Install API scaffolding (adds Sanctum)
php artisan install:api

# Configure .env with your DB credentials
# DB_DATABASE=task_manager
# DB_USERNAME=root
# DB_PASSWORD=

# Create the Task model, migration, and controller
php artisan make:model Task -mcr
Migration — database/migrations/xxxx_create_tasks_table.php:

php
public function up(): void
{
    Schema::create('tasks', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->cascadeOnDelete();
        $table->string('title');
        $table->text('description')->nullable();
        $table->boolean('completed')->default(false);
        $table->timestamps();
    });
}
Model — app/Models/Task.php:

php
class Task extends Model
{
    protected $fillable = ['title', 'description', 'completed'];
    protected $casts = ['completed' => 'boolean'];

    public function user()
    {
        return $this->belongsTo(User::class);
    }
}
Controller — app/Http/Controllers/TaskController.php:

php
class TaskController extends Controller
{
    public function index(Request $request)
    {
        return TaskResource::collection(
            $request->user()->tasks()->latest()->get()
        );
    }

    public function store(StoreTaskRequest $request)
    {
        $task = $request->user()->tasks()->create($request->validated());
        return new TaskResource($task);
    }

    public function update(UpdateTaskRequest $request, Task $task)
    {
        $this->authorize('update', $task);
        $task->update($request->validated());
        return new TaskResource($task);
    }

    public function destroy(Task $task)
    {
        $this->authorize('delete', $task);
        $task->delete();
        return response()->noContent();
    }
}
Routes — routes/api.php:

php
Route::middleware('auth:sanctum')->group(function () {
    Route::apiResource('tasks', TaskController::class);
});

Route::post('/register', [AuthController::class, 'register']);
Route::post('/login',    [AuthController::class, 'login']);
Run migrations and start the server:

bash
php artisan migrate
php artisan serve
API is now live at http://localhost:8000.

🅰️ Web Frontend Setup (Angular)
bash
ng new task-web
cd task-web
TaskService — src/app/services/task.service.ts:

typescript
@Injectable({ providedIn: 'root' })
export class TaskService {
  private api = 'http://localhost:8000/api';

  constructor(private http: HttpClient) {}

  getTasks() {
    return this.http.get<Task[]>(`${this.api}/tasks`);
  }

  create(task: Partial<Task>) {
    return this.http.post<Task>(`${this.api}/tasks`, task);
  }

  update(id: number, task: Partial<Task>) {
    return this.http.put<Task>(`${this.api}/tasks/${id}`, task);
  }

  delete(id: number) {
    return this.http.delete(`${this.api}/tasks/${id}`);
  }
}
Auth Interceptor — attaches the token to every request:

typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const token = localStorage.getItem('token');
  if (token) {
    req = req.clone({ setHeaders: { Authorization: `Bearer ${token}` } });
  }
  return next(req);
};
Register it in app.config.ts:

typescript
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(withInterceptors([authInterceptor])),
    provideRouter(routes),
  ],
};
Run the dev server:

bash
ng serve
📱 Mobile App Setup (Ionic)
bash
ionic start task-mobile blank --type=angular
cd task-mobile
Ionic reuses your Angular services and components — the TaskService and interceptor from the web app carry over directly.

Home Page — src/app/home/home.page.ts:

typescript
@Component({
  selector: 'app-home',
  template: `
    <ion-header>
      <ion-toolbar><ion-title>My Tasks</ion-title></ion-toolbar>
    </ion-header>
    <ion-content>
      <ion-list>
        <ion-item *ngFor="let task of tasks">
          <ion-checkbox slot="start" [checked]="task.completed"
                        (ionChange)="toggle(task)"></ion-checkbox>
          <ion-label>{{ task.title }}</ion-label>
        </ion-item>
      </ion-list>
    </ion-content>
  `,
})
export class HomePage {
  tasks: Task[] = [];

  constructor(private taskService: TaskService) {}

  ngOnInit() {
    this.taskService.getTasks().subscribe(t => (this.tasks = t));
  }
}
Run in browser:

bash
ionic serve
Run on device (requires Capacitor):

bash
ionic cap add android
ionic cap add ios
ionic cap run android
🔌 API Endpoints
Method	Endpoint	Auth	Description
POST	/api/register	❌	Register a new user
POST	/api/login	❌	Log in, returns Sanctum token
POST	/api/logout	✅	Revoke current token
GET	/api/tasks	✅	List authenticated user's tasks
POST	/api/tasks	✅	Create a new task
GET	/api/tasks/{id}	✅	Show a specific task
PUT	/api/tasks/{id}	✅	Update a task
DELETE	/api/tasks/{id}	✅	Delete a task
Example request:

bash
curl -X POST http://localhost:8000/api/tasks \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Learn Laravel","description":"Finish docs"}'
📁 Project Structure
text
task-manager/
├── task-api/                # Laravel backend
│   ├── app/
│   │   ├── Http/
│   │   │   ├── Controllers/
│   │   │   ├── Requests/
│   │   │   └── Resources/
│   │   ├── Models/
│   │   └── Policies/
│   ├── database/migrations/
│   ├── routes/api.php
│   └── .env
│
├── task-web/                # Angular web app
│   └── src/app/
│       ├── components/
│       ├── services/
│       ├── guards/
│       └── interceptors/
│
└── task-mobile/             # Ionic mobile app
    └── src/app/
        └── home/
📚 Learning Roadmap
Do not try to learn all four at once. Go in this order:

PHP basics — variables, arrays, functions, classes, PDO (1–2 weeks)

Laravel alone — build the task API. Test with Postman. No frontend yet. (2–3 weeks)

Angular — build the web frontend talking to your Laravel API. (2–3 weeks)

Ionic — port your Angular app to mobile. (1 week)

Refactor — add tests, resources, policies, form requests.

✅ Milestone Checklist
Backend (Laravel)
□ Register + login returns a Sanctum token
□ Authenticated user can CRUD their own tasks
□ Users cannot touch other users' tasks (policies)
□ Validation via Form Requests
□ JSON responses shaped via API Resources
Web Frontend (Angular)
□ Login form stores token and redirects to dashboard
□ Task list with create / edit / delete
□ Auth guard protects private routes
□ HTTP interceptor attaches token automatically
Mobile App (Ionic)
□ Same features work on mobile
□ Runs on emulator / physical device via Capacitor
💡 Tips for Learning
Start Laravel-only. Don't touch Angular until your API works in Postman.

Use php artisan route:list constantly — it teaches you routing fast.

Read the Laravel docs top-to-bottom once. They're the best docs in the PHP world.

Angular and Ionic share ~90% of code. Learn Angular first; Ionic is a small delta.

CORS gotcha: Laravel needs config/cors.php configured to allow your Angular dev server origin.

Postman / Insomnia are your best friends for testing API endpoints before building a UI.

Git commit often — one feature per commit makes debugging much easier.

📄 License
This project is for educational purposes. Feel free to use, modify, and learn from it
