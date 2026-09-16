# Project Detection Reference

Rules for detecting project stacks and recommending skills by category.

---

## Stack Detection

### Primary Indicators

| File / Pattern                      | Stack                                     | Priority |
| ----------------------------------- | ----------------------------------------- | -------- |
| `package.json`                      | Node.js (requires sub-detection)          | High     |
| `*.sln`                             | .NET Solution                             | High     |
| `*.csproj`                          | .NET Project (requires sub-detection)     | High     |
| `pom.xml`                           | Java (Maven)                              | High     |
| `build.gradle` / `build.gradle.kts` | Java/Kotlin (Gradle)                      | High     |
| `requirements.txt`                  | Python                                    | Medium   |
| `pyproject.toml`                    | Python                                    | High     |
| `setup.py` / `setup.cfg`            | Python                                    | Medium   |
| `go.mod`                            | Go                                        | High     |
| `Cargo.toml`                        | Rust                                      | High     |
| `composer.json`                     | PHP                                       | High     |
| `Gemfile`                           | Ruby                                      | High     |
| `Package.swift`                     | Swift                                     | High     |
| `pubspec.yaml`                      | Flutter / Dart                            | High     |
| `Makefile` only                     | C/C++ (tentative)                         | Low      |
| `CMakeLists.txt`                    | C/C++ (CMake)                             | Medium   |
| `angular.json`                      | Angular (even without package.json check) | High     |
| `next.config.*`                     | Next.js                                   | High     |
| `nuxt.config.*`                     | Nuxt.js                                   | High     |
| `vite.config.*`                     | Vite-based project                        | Medium   |
| `astro.config.*`                    | Astro                                     | High     |

### Node.js Sub-Detection (from package.json)

Read `dependencies` and `devDependencies` to determine the framework:

| Dependency                             | Sub-Stack           |
| -------------------------------------- | ------------------- |
| `react` / `react-dom`                  | React               |
| `next`                                 | Next.js (React)     |
| `@angular/core`                        | Angular             |
| `vue`                                  | Vue.js              |
| `nuxt`                                 | Nuxt (Vue)          |
| `svelte`                               | Svelte              |
| `@sveltejs/kit`                        | SvelteKit           |
| `solid-js`                             | Solid.js            |
| `astro`                                | Astro               |
| `express` / `fastify` / `koa` / `hapi` | Node.js Backend     |
| `@nestjs/core`                         | NestJS              |
| `electron`                             | Electron            |
| `react-native`                         | React Native        |
| `expo`                                 | Expo (React Native) |
| `@remix-run/node`                      | Remix               |

### .NET Sub-Detection (from .csproj)

Read the `.csproj` XML to identify the project type:

| Indicator                                                   | Sub-Stack                               |
| ----------------------------------------------------------- | --------------------------------------- |
| `<UseWindowsForms>true</UseWindowsForms>`                   | WinForms                                |
| `<UseWPF>true</UseWPF>`                                     | WPF                                     |
| `<Project Sdk="Microsoft.NET.Sdk.Web">`                     | ASP.NET (Web API / MVC / Blazor Server) |
| `<Project Sdk="Microsoft.NET.Sdk.BlazorWebAssembly">`       | Blazor WASM                             |
| `<UseMaui>true</UseMaui>`                                   | .NET MAUI                               |
| `PackageReference` includes `Microsoft.AspNetCore.*`        | ASP.NET                                 |
| `PackageReference` includes `Microsoft.EntityFrameworkCore` | EF Core present                         |
| `<OutputType>Exe</OutputType>` without Web SDK              | Console App                             |
| `<OutputType>Library</OutputType>`                          | Class Library                           |

### Python Sub-Detection

| Indicator                           | Sub-Stack          |
| ----------------------------------- | ------------------ |
| `django` in dependencies            | Django             |
| `flask` in dependencies             | Flask              |
| `fastapi` in dependencies           | FastAPI            |
| `pytorch` / `torch`                 | ML/AI (PyTorch)    |
| `tensorflow`                        | ML/AI (TensorFlow) |
| `pandas` / `numpy` / `scikit-learn` | Data Science       |
| `streamlit`                         | Streamlit          |

---

## Multi-Stack Detection

Projects can have multiple stacks. Common combinations:

| Indicators Found                                                | Detected As                     |
| --------------------------------------------------------------- | ------------------------------- |
| `.sln` + `package.json` with React                              | .NET + React (Full-stack)       |
| `.csproj` (Web) + `pubspec.yaml`                                | .NET Backend + Flutter Frontend |
| `package.json` (Express) + `package.json` (React) in subfolders | Node.js Full-stack              |
| `docker-compose.yml` + multiple stacks                          | Multi-service project           |
| `pom.xml` + `package.json` with Angular                         | Java + Angular (Full-stack)     |

When multiple stacks are detected, recommend skills from ALL matching categories.

---

## Recommendation Matrix

### Category → Stack Mapping

| Detected Stack            | Primary Categories                     | Secondary Categories                 |
| ------------------------- | -------------------------------------- | ------------------------------------ |
| React                     | `react/`, `frontend/`, `typescript/`   | `shared/`, `testing/`                |
| Next.js                   | `react/`, `nextjs/`, `frontend/`       | `shared/`, `testing/`, `deployment/` |
| Angular                   | `angular/`, `frontend/`, `typescript/` | `shared/`, `testing/`                |
| Vue.js / Nuxt             | `vue/`, `frontend/`                    | `shared/`, `testing/`                |
| .NET Web API              | `dotnet/`, `api/`, `backend/`          | `shared/`, `sql/`, `testing/`        |
| .NET WinForms             | `winforms/`, `dotnet/`, `desktop/`     | `shared/`, `sql/`                    |
| .NET WPF                  | `wpf/`, `dotnet/`, `desktop/`          | `shared/`, `sql/`                    |
| .NET Blazor               | `blazor/`, `dotnet/`, `frontend/`      | `shared/`, `sql/`, `testing/`        |
| .NET MAUI                 | `maui/`, `dotnet/`, `mobile/`          | `shared/`                            |
| Java (Spring)             | `java/`, `spring/`, `backend/`         | `shared/`, `sql/`, `testing/`        |
| Python (FastAPI)          | `python/`, `fastapi/`, `backend/`      | `shared/`, `sql/`, `testing/`        |
| Python (Django)           | `python/`, `django/`, `backend/`       | `shared/`, `sql/`, `testing/`        |
| Python (ML/AI)            | `python/`, `ml/`, `data-science/`      | `shared/`                            |
| Go                        | `go/`, `backend/`                      | `shared/`, `testing/`                |
| Rust                      | `rust/`                                | `shared/`, `testing/`                |
| Flutter                   | `flutter/`, `dart/`, `mobile/`         | `shared/`                            |
| Full-stack (.NET + React) | `dotnet/`, `react/`, `frontend/`       | `shared/`, `sql/`, `testing/`        |

### Priority Rules

1. **Exact category match** (e.g., `react/` for React projects) → ★★★ highest priority
2. **Related category** (e.g., `frontend/` for React) → ★★ medium priority
3. **Universal category** (e.g., `shared/`, `testing/`) → ★ always recommend
4. **Unrelated category** (e.g., `dotnet/` for a React project) → listed but not recommended

### Presentation Order

1. Show ★★★ items first with strong recommendation
2. Show ★★ items as "also recommended"
3. Show ★ items as "generally useful"
4. Show remaining items as "also available" (collapsed/secondary)
