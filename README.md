# C# DDD AI Project Creator

An AI-powered onboarding guide for creating and validating Domain-Driven Design (DDD) .NET 9 solutions with layered architecture, Autofac DI, and comprehensive test projects.

## 🎯 What This Project Does

This repository contains an **AI agent onboarding guide** (`onboarding_guide.md`) that systematically transforms empty or starter .NET projects into fully compliant DDD layered architecture solutions following best practices.

### What the Onboarding Guide Does

The `onboarding_guide.md` file serves as a comprehensive instruction manual for AI agents (like Claude, ChatGPT, etc.) to:

1. **Validate existing project structure** - Checks for solution files, correct project naming, and proper DDD layer organization
2. **Enforce architectural rules** - Ensures Domain-Driven Design principles with proper dependency flow (Presentation → Application → Domain, Infrastructure → Domain)
3. **Configure projects correctly** - Sets up .NET 9 target frameworks, WPF configuration, nullable reference types, and platform targets
4. **Install required NuGet packages** - Manages dependencies with version ranges for maintainability
5. **Create test projects** - Generates NUnit test projects for all layers with proper configuration
6. **Set up Autofac DI** - Implements dependency injection with module-per-layer pattern and assembly scanning
7. **Implement MVVM patterns** - Creates ViewModels, Views, and proper data binding using DevExpress and MahApps.Metro
8. **Generate boilerplate code** - Scaffolds initial verification tests, modules, and configuration files

## 📚 Libraries Used in Projects

### Presentation Layer (WPF Application)

- **Autofac** (8.*) - Dependency injection container
- **Autofac.Extensions.DependencyInjection** (10.*) - Microsoft DI integration
- **DevExpressMvvm** (*) - MVVM framework with ViewModelBase and commands
- **MahApps.Metro** (2.*) - Modern Metro-style UI framework
- **Microsoft.Extensions.Logging** (8.*) - Logging abstractions
- **Microsoft.Extensions.Logging.Abstractions** (8.*) - Logging interfaces
- **NLog** (6.*) - Logging implementation
- **NLog.Extensions.Logging** (6.*) - NLog integration with Microsoft.Extensions.Logging
- **System.Reactive** (6.*) - Reactive Extensions for event-driven programming

### Application Layer

- **Autofac** (8.*) - Dependency injection
- **Autofac.Extensions.DependencyInjection** (10.*) - DI integration
- **NLog** (6.*) - Logging
- **NLog.Extensions.Logging** (6.*) - Logging integration

### Infrastructure Layer

- **Autofac** (8.*) - Dependency injection
- **Autofac.Extensions.DependencyInjection** (10.*) - DI integration
- **NLog** (6.*) - Logging
- **NLog.Extensions.Logging** (6.*) - Logging integration
- **System.Reactive** (6.*) - Reactive Extensions

### Domain Layer

- **No dependencies** - Domain layer remains pure with no external NuGet packages

## 🧪 Libraries Used for Tests

All test projects use the following NUnit-based testing stack:

- **AutoFixture** (4.*) - Auto-generating test data
- **AutoFixture.AutoMoq** (4.*) - Auto-mocking with Moq integration
- **Microsoft.Bcl.TimeProvider** (9.*) - Time abstraction for testing
- **Microsoft.Extensions.TimeProvider.Testing** (9.*) - Fake time provider for tests
- **Microsoft.Reactive.Testing** (6.*) - Testing utilities for Reactive Extensions
- **Moq** (4.*) - Mocking framework
- **NUnit** (4.*) - Testing framework
- **NUnit3TestAdapter** (5.*) - Visual Studio test adapter
- **Microsoft.NET.Test.Sdk** (17.*) - .NET test SDK
- **NUnit.Analyzers** (4.*) - Code analyzers for NUnit best practices

## 🚀 How to Use This Guide

### For AI Agents

1. **Place `onboarding_guide.md` in your project folder** (root directory of your .NET solution)
2. **Paste `race_prompt.md` to your AI assistant** (Claude, ChatGPT, etc.)
3. The AI will:
   - Read the onboarding guide
   - Validate your existing project structure
   - Create missing projects and test projects
   - Install required NuGet packages
   - Set up Autofac modules
   - Implement MVVM patterns
   - Document any errors in `errors.md`

## 📊 Project Structure Overview

### Before Onboarding Guide

```plaintext
YourProject/
├── YourProject.sln
└── (empty or incomplete project structure)
```

### After Onboarding Guide

```plaintext
YourProject/
├── YourProject.sln
├── .gitignore
├── .editorconfig
├── YourProject/ (Presentation - WPF)
│   ├── App.xaml
│   ├── MainWindow.xaml
│   ├── ViewModels/
│   ├── Modules/PresentationModule.cs
│   └── NLog.config
├── YourProjectApplication/
│   └── Modules/ApplicationModule.cs
├── YourProjectInfrastructure/
│   └── Modules/InfrastructureModule.cs
├── YourProjectDomain/
│   └── Content/
├── YourProject.Tests/
│   └── InitialVerificationTests.cs
├── YourProjectApplication.Tests/
│   └── InitialVerificationTests.cs
├── YourProjectInfrastructure.Tests/
│   └── InitialVerificationTests.cs
└── YourProjectDomain.Tests/
    └── InitialVerificationTests.cs
```

![Project structure before onboarding guide](docs/project-before.png)
![Project structure after onboarding guide](docs/project-after.png)

## 📐 Dependency Diagram

The guide enforces proper DDD layer dependencies:

```plaintext
Presentation Layer (WPF)
    │
    ├───> Application Layer ────┐
    │                           │
    └───> Infrastructure Layer ─┤
                                │
                                ▼
                         Domain Layer
                      (No Dependencies)
```

![Project dependency diagram](docs/dependency-diagram.png)

## 🐛 Error Tracking with errors.md

### What is errors.md?

The `errors.md` file is an **error log and learning document** that the AI agent creates during the onboarding process. It serves multiple purposes:

1. **Error Documentation** - Records every error encountered during project transformation
2. **Resolution History** - Documents how each error was resolved
3. **Guide Improvement** - Identifies gaps or unclear sections in the onboarding guide
4. **Troubleshooting Reference** - Helps developers understand common issues

### Structure of errors.md

Each error entry contains:

- **Phase** - Which onboarding phase the error occurred in
- **Guide Section** - Reference to the specific guide section
- **Severity** - Critical, Warning, or Info
- **What Failed** - Description of the error
- **Expected State** - What the guide requires
- **Actual State** - What was found in the project
- **Root Cause** - Why the error happened
- **Resolution Taken** - Step-by-step fix
- **Verification** - How the fix was confirmed
- **Guide Improvement Suggestion** - Recommendations for making the guide clearer

### How to Use errors.md for Troubleshooting

#### 1. **During AI Onboarding**

The AI automatically creates and updates `errors.md` as it encounters issues. You can monitor this file in real-time to see what problems are being fixed.

#### 2. **After Completion**

Review `errors.md` to understand:

- What problems were automatically fixed
- How the AI resolved each issue
- Whether any manual intervention is needed

#### 3. **For Similar Projects**

Use `errors.md` as a reference for common issues in your environment (e.g., network restrictions, missing SDKs, corporate proxy settings).

#### 4. **To Improve the Guide**

Submit suggestions from the "Guide Improvement Suggestion" sections to improve the onboarding guide for future projects.

### Example Troubleshooting Workflow

If the AI agent encounters an error during transformation:

```plaintext
1. AI detects: "Missing NuGet package 'Autofac' in Presentation layer"

2. AI writes to errors.md:
   ### Error 1: Missing Autofac Package in Presentation Layer
   
   **Phase:** Phase 3 - NuGet Packages Installation
   **Guide Section:** Presentation Layer Requirements > Required NuGet Packages
   **Severity:** Critical
   
   **What Failed:** 
   Package 'Autofac' not found in WpfAppExample.csproj
   
   **Expected State (per guide):**
   Autofac (Version="8.*") must be installed in Presentation layer
   
   **Actual State:**
   No Autofac package reference found
   
   **Root Cause:**
   Starter project template doesn't include dependency injection packages
   
   **Resolution Taken:**
   Ran: dotnet add WpfAppExample package Autofac --version "8.*"
   
   **Verification:**
   Package reference added to .csproj file
   dotnet build succeeded
   
   **Guide Improvement Suggestion:**
   Add note about expected delay when installing packages for first time

3. AI continues with next step

4. You review errors.md to see what was automatically fixed
```

### When to Check errors.md

- **Build fails** - Check if a similar error was already encountered and how it was resolved
- **Test failures** - Look for test-related errors and their resolutions
- **Package conflicts** - Find version-related issues and how they were fixed
- **Structural issues** - Understand project organization problems and their fixes
- **After transformation** - Review the complete transformation history
- **Before asking for help** - Check if your issue is already documented with a solution

### Common Error Categories in errors.md

The `errors.md` file typically documents issues in these categories:

1. **Missing Dependencies** - NuGet packages not installed
2. **Incorrect Target Frameworks** - Framework version mismatches
3. **Project Reference Issues** - Circular or missing references
4. **Configuration Problems** - .csproj settings, platform targets
5. **Module Registration Errors** - Autofac DI configuration issues
6. **MVVM Binding Errors** - DataContext, command binding problems
7. **Build Failures** - Compilation errors and their resolutions

## 📝 Files in This Repository

- **`onboarding_guide.md`** - Complete AI agent instruction manual (comprehensive DDD transformation guide)
- **`race_prompt.md`** - RACE-formatted prompt to activate the AI agent
- **`errors.md`** - Example error log from successful WpfAppExample transformation
- **`errors_template.md`** - Template for error documentation format
- **`WpfAppExample/`** - Example project demonstrating the final structure after transformation

## 📄 License

This project is provided as-is for educational and development purposes.

---

**Note:** This is a meta-project - the guide itself is the product. Use it to bootstrap DDD .NET 9 projects with AI assistance.
