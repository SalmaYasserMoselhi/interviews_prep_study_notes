# Python Refresher → Django REST Framework → PostgreSQL Roadmap

Sources: [docs.python.org](https://docs.python.org/3/), [docs.djangoproject.com](https://docs.djangoproject.com/en/6.0/) (Django 6.0), [django-rest-framework.org](https://www.django-rest-framework.org/), [postgresql.org/docs](https://www.postgresql.org/docs/).
Pairs with `react-nextjs-roadmap.md`. Together they cover both ends of a full-stack SDLC: React/Next.js on the client, Django REST + PostgreSQL on the server.

---

## PART 1 — PYTHON REFRESHER

You don't need to relearn Python from zero, but Django leans hard on a specific subset of "advanced-feeling but actually core" Python. Get fluent in this before touching Django, or every Django concept will feel like learning two things at once.

### 1. Core Language Refresher
- Variables, types, operators, control flow (`if`/`elif`/`else`, `for`, `while`, `match`/`case`)
- Data structures: `list`, `tuple`, `dict`, `set`, and their comprehensions
- Strings: f-strings, slicing, common methods
- Functions: default args, `*args`/`**kwargs`, keyword-only args, first-class functions
- Exception handling: `try`/`except`/`else`/`finally`, custom exceptions, exception chaining

### 2. Object-Oriented Python (Django is OOP top to bottom)
- Classes, `__init__`, instance vs. class attributes
- Inheritance, `super()`, method resolution order (MRO)
- **Magic/dunder methods**: `__str__`, `__repr__`, `__eq__`, `__len__`, `__call__` — Django models and querysets use these constantly
- `@property`, `@staticmethod`, `@classmethod`
- Abstract base classes (`abc` module) — relevant once you hit abstract Django models

### 3. Functional & Intermediate Features
- Decorators (writing your own, and understanding `@login_required`, `@api_view`, `@action` before you ever see them in DRF)
- Context managers (`with`, `__enter__`/`__exit__`) — needed to understand `transaction.atomic()`
- Generators and `yield` (relevant for queryset iteration and large dataset processing)
- `itertools` and `functools` basics (`reduce`, `partial`, `lru_cache`)
- Type hints (`typing` module): function annotations, `Optional`, `Union`/`|`, generics — DRF and modern Django code leans on these increasingly

### 4. Modules, Packages & Environment
- Imports, packages, `__init__.py`, relative vs. absolute imports
- Virtual environments (`venv`), `pip`, `requirements.txt` / `pyproject.toml`
- Reading and writing a `pyproject.toml` (modern packaging) — at minimum enough to configure tooling
- Environment variables (`os.environ`) and `.env` file patterns (you'll use this immediately for `SECRET_KEY`, `DATABASE_URL`)

### 5. Async Python (Django and DRF both increasingly support this)
- `async`/`await`, coroutines, the event loop conceptually
- When Django actually benefits from async views vs. when it doesn't (I/O-bound vs. CPU-bound)

### 6. Tooling You'll Use Daily
- `pytest` basics (fixtures, parametrize) — DRF testing leans on this ecosystem heavily even though Django ships `unittest`-style tests by default
- Linting/formatting: `ruff` or `flake8` + `black`
- Debugging: `pdb`/`breakpoint()`, and reading tracebacks fluently

✅ **Checkpoint:** comfortable reading and writing a class hierarchy with custom dunder methods, a decorator, a context manager, and a generator, without having to look any of them up.

---

## PART 2 — DJANGO (the full framework, nothing skipped)

> Django's own docs split into four kinds of pages: **Tutorials** (hands-on), **Topic guides** (concepts), **Reference** (API detail), **How-to guides** (recipes). The order below moves through topic guides in dependency order, pulling in reference/how-to material at the point it's needed — this mirrors how docs.djangoproject.com itself groups things on its homepage.

### 7. First Steps
- Installing Django, creating a project vs. an app (`django-admin startproject` / `manage.py startapp`)
- Project structure: `settings.py`, `urls.py`, `asgi.py`, `wsgi.py`, `manage.py`
- The dev server, and why it's explicitly **not** for production
- Django's request/response cycle, end to end (this mental model underlies everything else)

### 8. Settings & Configuration
- `settings.py` structure, environment-based settings (dev/staging/prod split)
- `INSTALLED_APPS`, what registering an app actually does
- The **full settings reference** — at minimum know it exists and skim it once: `DEBUG`, `ALLOWED_HOSTS`, `DATABASES`, `SECRET_KEY`, `STATIC_URL`, `MIDDLEWARE`, `TIME_ZONE`, `USE_TZ`
- **Applications** — the `AppConfig` system, `apps.py`, app registry

### 9. The Model Layer (this is where PostgreSQL/ORM enters — see Section 9.5)
- **Models**: defining models, field types (`CharField`, `IntegerField`, `ForeignKey`, `ManyToManyField`, `JSONField`, etc.), `Meta` options, model methods
- **Model class internals**: instance methods, accessing related objects (`related_name`, reverse relations)
- **Migrations**: what they are, `makemigrations`/`migrate`, the migration graph, writing migrations by hand, `SchemaEditor`
- **Making queries**: the QuerySet API — filtering, chaining, `Q` objects, `F` expressions, lazy evaluation
- **Aggregation** (`annotate`, `aggregate`, `Count`, `Sum`, `Avg`)
- **Search** (basic `contains`/`icontains` lookups vs. Postgres full-text search — covered in 9.5)
- **Managers** — custom managers, `objects`, chaining manager methods
- **Raw SQL** — `.raw()`, `connection.cursor()`, and *when* dropping to raw SQL is justified
- **Transactions** — `transaction.atomic()`, autocommit, savepoints, why this matters for data integrity
- **Multiple databases** — routing reads/writes across DBs (good to know exists even if you don't need it day one)
- **Custom fields**, **custom lookups**, **Query Expressions**, **Conditional Expressions**, **Database Functions** — advanced ORM toolkit, come back to these once you're comfortable with the basics
- **Database access optimization** — `select_related` vs. `prefetch_related`, avoiding N+1 queries, `only()`/`defer()`, indexing strategy (ties directly into PostgreSQL `EXPLAIN`, Section 9.5)
- **Fixtures** — loading initial/test data
- **Supported databases**, **legacy databases** — context, not depth

#### 9.5 PostgreSQL Refresher + Django's ORM as your O*RM* layer
*(This is the deliberate checkpoint where SQL/Postgres knowledge and the Django ORM meet. Don't skip the raw SQL side — knowing what the ORM generates under the hood is what separates devs who can debug slow queries from devs who can't.)*

**PostgreSQL fundamentals (if rusty):**
- Relational model basics: tables, rows, columns, primary/foreign keys, normalization (1NF–3NF) and when to deliberately denormalize
- `CREATE TABLE`, data types (`INTEGER`, `TEXT`, `VARCHAR`, `BOOLEAN`, `TIMESTAMP`, `JSONB`, `UUID`, `ARRAY`)
- `SELECT`/`INSERT`/`UPDATE`/`DELETE`, `WHERE`, `ORDER BY`, `LIMIT`/`OFFSET`
- `JOIN` types: `INNER`, `LEFT`, `RIGHT`, `FULL OUTER` — and how each maps to Django's `ForeignKey`/`ManyToManyField` traversal
- `GROUP BY` + aggregate functions — maps directly to Django's `.annotate()`/`.aggregate()`
- Constraints: `NOT NULL`, `UNIQUE`, `CHECK`, `FOREIGN KEY ... ON DELETE` (maps to Django's `on_delete=models.CASCADE/PROTECT/SET_NULL`, etc.)
- Indexes: B-tree by default, when to add one, composite indexes, partial indexes
- Transactions and isolation levels (`BEGIN`/`COMMIT`/`ROLLBACK`, read committed vs. serializable) — the database-level concept underneath `transaction.atomic()`
- `EXPLAIN` / `EXPLAIN ANALYZE` — reading a query plan, spotting sequential scans that should be index scans

**Where the Django ORM and Postgres meet:**
- Turn on `django.db.backends` logging (or use Django Debug Toolbar) and *look at the actual SQL* your ORM calls generate — do this early and often
- **N+1 queries**: what they are, why `select_related` (SQL JOIN, for FK/O2O) and `prefetch_related` (separate query + Python-side join, for M2M/reverse FK) solve them differently
- Migrations as version-controlled schema changes — what Django generates vs. what you sometimes need to hand-edit (e.g., adding a `NOT NULL` column to a populated table safely, data migrations)
- **PostgreSQL-specific Django features** (`django.contrib.postgres`): `ArrayField`, `JSONField` (native Postgres JSONB), `HStoreField`, full-text search (`SearchVector`, `SearchQuery`, `SearchRank`), `TrigramSimilarity`, range fields, and Postgres-specific constraints/indexes (`GinIndex`, exclusion constraints)
- Connection pooling concepts (`pgbouncer`) and why Django's default per-request connection behavior matters at scale
- Backing up/restoring (`pg_dump`/`pg_restore`) — operational basics every backend dev should have run at least once

✅ **Checkpoint for this section:** given a slow Django view, you should be able to find the generated SQL, run `EXPLAIN ANALYZE` on it, identify whether it's an N+1 problem or a missing index, and fix it with the correct ORM construct (not just "throw more caching at it").

### 10. The View Layer
- **URLconfs**: `path()`, `re_path()`, `include()`, named URL patterns, the URL namespace system
- **View functions**: writing function-based views, `HttpRequest`/`HttpResponse`
- **Shortcuts**: `render()`, `redirect()`, `get_object_or_404()`
- **Decorators**: `@login_required`, `@require_http_methods`, caching decorators
- **Class-based views**: `View`, generic display views (`ListView`/`DetailView`), generic editing views (`CreateView`/`UpdateView`/`DeleteView`), mixins, when CBVs help vs. hurt readability
- **Built-in views, Request/Response objects, TemplateResponse** — reference-level familiarity
- **File uploads**: handling uploaded files, `File` objects, the Storage API, custom storage backends
- **Middleware**: what middleware is, the request/response middleware chain, writing custom middleware, built-in middleware classes (security, sessions, CSRF, etc.)
- **Asynchronous views** — `async def` views, when they help (high-concurrency I/O-bound endpoints)

### 11. The Template Layer
*(You're building an API with DRF, so templates matter less day-to-day — but you need them for: the Django admin, the DRF browsable API, error pages, and any server-rendered emails.)*
- Template basics, the template language (tags, filters, template inheritance)
- Custom template tags and filters
- Template API for programmatic rendering

### 12. Forms
*(Same caveat as templates — DRF replaces most of this for your API surface, but you still need the concepts.)*
- Form basics, built-in fields and widgets
- ModelForms (the conceptual ancestor of DRF's `ModelSerializer` — understanding this makes DRF serializers click faster)
- Formsets, validation customization

### 13. The Development Process
- **Applications** (`AppConfig`) — revisit once you're structuring a multi-app project
- **Exceptions** — Django's built-in exception hierarchy (`Http404`, `PermissionDenied`, `ValidationError`, etc.)
- **`django-admin` and `manage.py`** — built-in commands, writing custom management commands (essential for one-off scripts, data backfills, scheduled jobs)
- **Testing**: writing and running tests, `TestCase` vs. `TransactionTestCase`, the test client, fixtures in tests, included testing tools, advanced topics (mocking, overriding settings)
- **Deployment**: WSGI vs. ASGI servers (`gunicorn`/`uvicorn`), serving static files in production, error reporting/email alerts, and — critically — running through the **deployment checklist** before any real launch

### 14. The Admin
- The automatic admin interface: registering models, `ModelAdmin` customization (`list_display`, `search_fields`, `list_filter`, inlines)
- Admin actions (bulk operations on selected objects)
- Admin documentation generator

### 15. Security
*(Don't treat this as optional reading — this is the section that turns a working API into a safe one.)*
- Django's built-in protections and what they cover by default
- Clickjacking protection (`X-Frame-Options`)
- **CSRF protection** — how it works, and why it matters differently for session-auth vs. token-auth APIs
- Cryptographic signing (`django.core.signing`)
- Security middleware (`SECURE_*` settings: HSTS, SSL redirect, content-type sniffing protection)
- Content Security Policy (CSP)
- SQL injection, XSS — how the ORM and template engine protect you by default, and the specific ways you can accidentally defeat that protection (raw SQL, `mark_safe`, `|safe`)

### 16. Internationalization & Localization
- i18n vs. l10n, translation files (`.po`/`.mo`), `gettext`
- Localized formatting (dates, numbers) and form input
- Time zones — `USE_TZ`, storing UTC vs. displaying local time (this one bites everyone eventually)

### 17. Performance & Optimization
- Django's own performance overview: where to look first (DB queries, template rendering, middleware overhead)
- Ties directly back into Section 9.5 (query optimization) — this section is the "now zoom out to the whole app" version of that

### 18. Common Web Application Tools (Django's "batteries included" layer)
- **Authentication**: the built-in auth system overview, `User` model vs. custom user models (**decide this at project start — swapping later is painful**), permissions and groups, password management/hashing, customizing authentication backends
- **Caching**: cache backends, per-view caching, low-level cache API, template fragment caching
- **Logging**: configuring Django's logging, where to actually put log calls
- **Tasks framework** — Django's task/queue-adjacent tooling (and where you'd reach for Celery/RQ instead for heavier async job needs)
- **Sending email**
- **Syndication feeds (RSS/Atom)**
- **Pagination** — Django's built-in `Paginator` (note: DRF has its own pagination layer you'll likely use instead for the API, but know this exists)
- **Messages framework** — flash-style messages (mostly template-rendered apps, but good to know)
- **Serialization** — Django's built-in serializers (`dumpdata`/`loaddata`); contrast with DRF serializers later
- **Sessions** — session backends, session security
- **Sitemaps**, **static files management**, **data validation (`django.core.validators`)**

### 19. Other Core Functionalities
- Conditional content processing (`ETag`/`Last-Modified` support)
- **Content types and generic relations** (`ContentType` framework — needed for things like a generic "comments on anything" or "tags on anything" model)
- Flatpages, redirects (`django.contrib.redirects`)
- **Signals** — `post_save`, `pre_delete`, etc., and the case for/against using them (they're powerful and easy to overuse into invisible side effects)
- System check framework (`manage.py check`)
- The sites framework
- Unicode handling in Django

✅ **Checkpoint:** you should be able to stand up a Django project with a custom user model, proper settings split, the admin fully configured for your models, signals used deliberately (not by default), and a passing test suite — entirely without DRF yet. That's "knows Django." Next is "knows how to expose it as an API."

---

## PART 3 — DJANGO REST FRAMEWORK

> DRF's own docs are organized as: a 6-part **Tutorial**, a reference-heavy **API Guide**, and conceptual **Topics**. Go through the tutorial once start to finish for the narrative, then treat the API Guide as the thing you actually master topic by topic.

### 20. DRF Fundamentals (the Tutorial, in order)
1. **Serialization** — `Serializer`, converting model instances to/from native Python types and JSON
2. **Requests and Responses** — DRF's `Request`/`Response` objects vs. Django's raw `HttpRequest`/`HttpResponse`
3. **Class-based views** — `APIView`
4. **Authentication and permissions** — wiring up your first protected endpoint
5. **Relationships and hyperlinked APIs** — `PrimaryKeyRelatedField` vs. `HyperlinkedModelSerializer`
6. **Viewsets and routers** — collapsing repetitive CRUD views into `ModelViewSet` + `DefaultRouter`

### 21. Serializers (deep dive)
- `Serializer` vs. `ModelSerializer` (the DRF analog to Django's `ModelForm` — this is why Section 12 mattered)
- **Serializer fields** — the full field type catalog, `source=`, `read_only`/`write_only`
- **Serializer relations** — nested serializers, `PrimaryKeyRelatedField`, `SlugRelatedField`, `StringRelatedField`, writable nested serializers (and why they're genuinely tricky)
- Validation: field-level (`validate_<field>`), object-level (`validate`), and **Validators** (reusable validator classes/functions, `UniqueTogetherValidator`)
- `to_representation`/`to_internal_value` overrides for custom serialization logic

### 22. Views, Generic Views & ViewSets
- `APIView` — the base building block
- **Generic views** — `GenericAPIView` + mixins (`ListModelMixin`, `CreateModelMixin`, etc.), and the concrete generic views built from them (`ListCreateAPIView`, `RetrieveUpdateDestroyAPIView`, ...)
- **Viewsets** — `ViewSet`, `GenericViewSet`, `ModelViewSet`, `ReadOnlyModelViewSet`; overriding `get_queryset()`/`get_serializer_class()`/`perform_create()` per-action
- Custom actions with `@action` (`detail=True/False`) for endpoints beyond standard CRUD
- **Routers** — `SimpleRouter` vs. `DefaultRouter`, manual URL wiring as the escape hatch when routers aren't flexible enough

### 23. Authentication & Permissions (the part that makes an API actually safe to ship)
- **Authentication**: `SessionAuthentication`, `BasicAuthentication`, `TokenAuthentication`, and JWT-based auth (`djangorestframework-simplejwt` or similar) — know the tradeoffs between session cookies and bearer tokens for an API consumed by a separate frontend
- **Permissions**: `AllowAny`, `IsAuthenticated`, `IsAdminUser`, `IsAuthenticatedOrReadOnly`, `DjangoModelPermissions`, `DjangoObjectPermissions`
- **Custom permission classes** — `has_permission()` vs. `has_object_permission()`, the classic `IsOwnerOrReadOnly` pattern
- Composing permissions with `&`, `|`, `~`
- Understanding the three independent layers of access control and what each one can/can't restrict: `queryset`/`get_queryset()` (visibility), `permission_classes`/`get_permissions()` (action-level), `serializer_class`/`get_serializer()` (field-level) — this table is worth memorizing, it's where most "wait, why can they still see that" bugs come from

### 24. Filtering, Pagination, Throttling & Caching
- **Filtering**: `filter_backends`, `DjangoFilterBackend` (`django-filter`), `SearchFilter`, `OrderingFilter`
- **Pagination**: `PageNumberPagination`, `LimitOffsetPagination`, `CursorPagination` — and when cursor pagination is the only correct choice (large, frequently-changing datasets)
- **Throttling**: rate-limiting by user/anon/scope
- **Caching**: response caching strategies for read-heavy endpoints

### 25. The Rest of the API Guide (round out full mastery)
- **Requests/Responses** reference depth — content negotiation details, `request.data` vs. `request.POST`
- **Parsers** and **Renderers** — how DRF decides how to parse incoming bodies and render outgoing responses (JSON by default, but know the extension points: `BrowsableAPIRenderer`, custom renderers for CSV/XML/etc.)
- **Versioning** — URL path versioning, header versioning, and picking one deliberately at project start
- **Content negotiation** — how `Accept` headers map to renderers
- **Metadata** — `OPTIONS` request handling
- **Schemas** — OpenAPI schema generation (`drf-spectacular` is the modern standard; built-in `AutoSchema` as the baseline) — this is what powers auto-generated API docs and client SDK generation
- **Format suffixes** (`.json`, `.api`) as an alternative to content negotiation
- **Returning URLs** (`reverse()` in a DRF context)
- **Exceptions** and **Status codes** — DRF's exception handling pipeline, the full HTTP status code reference, writing custom exception handlers
- **Testing** — `APIClient`, `APITestCase`, testing authenticated requests, testing permission boundaries explicitly (not just happy paths)
- **Settings** — the full `REST_FRAMEWORK` settings dict and what each default actually controls

### 26. DRF Topics (the conceptual layer beyond the API Guide)
- Documenting your API (schema-driven docs, browsable API as living documentation)
- Internationalization in DRF
- AJAX, CSRF & CORS — **CORS specifically matters a lot for you**, since your frontend (Next.js, on a different origin during development and likely production) needs `django-cors-headers` configured correctly or every request silently fails
- HTML & Forms, Browser Enhancements, the Browsable API — what it is and why it's a genuinely useful dev tool, not just a demo feature
- REST, Hypermedia & HATEOAS — the theoretical backdrop, worth understanding even if you don't build a fully hypermedia-driven API

✅ **Checkpoint:** you should be able to design a DRF API for a multi-model domain (entities with ownership, relationships, and role-based access) from scratch: models → migrations → serializers (including nested/writable relations) → viewsets with per-action queryset/permission/serializer logic → routers → auth → filtering/pagination → tests that explicitly check both "the right people can" and "the wrong people can't."

---

## PART 4 — TYING IT TOGETHER (SDLC awareness)

With both this roadmap and the React/Next.js one done, the full-stack SDLC loop looks like:

1. **Design** — data model (Postgres schema) → API contract (DRF serializers/endpoints, ideally schema-generated docs) → frontend data needs (what the Next.js Server Components will fetch)
2. **Build** — Django models/migrations → DRF viewsets/permissions → React/Next.js consuming the API (Server Components for reads, Server Actions or client `fetch` to DRF endpoints for writes)
3. **Test** — DRF `APITestCase` for the backend contract, React Testing Library for components, and end-to-end tests across both
4. **Secure** — CORS configured correctly, auth strategy chosen deliberately (session vs. JWT) and consistent across both apps, Django's security middleware + CSP, input validation at the serializer layer (never trust the frontend's validation alone)
5. **Deploy** — Django behind `gunicorn`/`uvicorn` + a reverse proxy, Postgres as a managed/separate service, Next.js on Vercel or self-hosted, environment variables/secrets managed per environment
6. **Operate** — logging, error tracking, query performance monitoring (the `EXPLAIN ANALYZE` habit from Section 9.5 never stops mattering), and the cache/revalidation story coordinated on both sides (Next.js `revalidateTag` triggered by, e.g., a Django signal or webhook after a mutation)

---

## Quick reference: official doc entry points
- Python docs: https://docs.python.org/3/
- Django documentation home: https://docs.djangoproject.com/en/6.0/
- Django topic guides: https://docs.djangoproject.com/en/6.0/topics/
- Django REST Framework: https://www.django-rest-framework.org/
- DRF Tutorial: https://www.django-rest-framework.org/tutorial/quickstart/
- DRF API Guide: https://www.django-rest-framework.org/api-guide/requests/
- PostgreSQL documentation: https://www.postgresql.org/docs/
