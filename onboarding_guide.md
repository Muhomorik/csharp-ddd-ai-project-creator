# 🤖 AI Agent OnBoarding & Execution Guide
> Comprehensive instructions for AI agents to validate, recreate, and work with Domain-Driven Design .NET 9 solutions using Autofac DI and layered architecture.

---

## 📋 MUST READ: Solution & Project Structure Validation

**REQUIRED BEFORE generating code:**
- Confirm the solution file (`.sln`) exists in the repository root.
- Confirm all four projects exist using the placeholder mapping:
  - `[MyProject.Presentation]` → Main project (e.g., `MyfancyProject`)
  - `[MyProject.Application]` → Application layer (e.g., `MyfancyProjectApplication`)
  - `[MyProject.Infrastructure]` → Infrastructure layer (e.g., `MyfancyProjectInfrastructure`)
  - `[MyProject.Domain]` → Domain layer (e.g., `MyfancyProjectDomain`)
- If all four projects exist, **break execution** and output:  
  _"Build stopped: All four projects exist in the solution."_
- If the solution file is missing, **break execution** and output:  
  _"Build stopped: Solution file is missing."_

---

## 🏗️ Project Placeholder Mapping Convention

In documentation and scripts, the following placeholders represent project layers:

- `[MyProject.Presentation]` → `<ProjectName>` (WPF Application)
- `[MyProject.Application]` → `<ProjectName>Application`
- `[MyProject.Infrastructure]` → `<ProjectName>Infrastructure`
- `[MyProject.Domain]` → `<ProjectName>Domain`

**Example Mapping:**
If your project name is `MyfancyProject`:
- `[MyProject.Presentation]` → `MyfancyProject`
- `[MyProject.Application]` → `MyfancyProjectApplication`
- `[MyProject.Infrastructure]` → `MyfancyProjectInfrastructure`
- `[MyProject.Domain]` → `MyfancyProjectDomain`

---

## ⚠️ REQUIRED: Layered Architecture & Dependencies

**Domain-Driven Design (DDD) Layering:**
- Presentation → Application → Domain
- Presentation → Infrastructure → Domain
- Infrastructure → Domain

**Validate:**
- Each project only references allowed layers.
- No UI types in Application/Domain.
- Infrastructure implements Domain interfaces only.

---

## 🔧 Autofac Module Configuration

### **Module per Project Pattern**
Each project must have its own Autofac module with naming convention:
- **Domain:** `[ProjectName]DomainModule` (optional - typically just entities)
- **Infrastructure:** `[ProjectName]InfrastructureModule` 
- **Application:** `[ProjectName]ApplicationModule`
- **Presentation:** `[ProjectName]PresentationModule`

### **Required Module Structure**

#### Infrastructure Module Example:
```csharp
// [MyProject.Infrastructure]/Modules/InfrastructureModule.cs
using Autofac;

namespace [MyProject.Infrastructure].Modules
{
    public class InfrastructureModule : Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            // Register implementations of domain interfaces
            builder.RegisterAssemblyTypes(typeof(InfrastructureModule).Assembly)
                   .Where(t => t.Name.EndsWith("Repository") || 
                              t.Name.EndsWith("Service"))
                   .AsImplementedInterfaces()
                   .InstancePerLifetimeScope();
                   
            // Manual registrations for specific cases
            // builder.RegisterType<UserRepository>()
            //        .As<IUserRepository>()
            //        .InstancePerLifetimeScope();
        }
    }
}
```

#### Application Module Example:
```csharp
// [MyProject.Application]/Modules/ApplicationModule.cs
using Autofac;

namespace [MyProject.Application].Modules
{
    public class ApplicationModule : Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            // Register application services
            builder.RegisterAssemblyTypes(typeof(ApplicationModule).Assembly)
                   .Where(t => t.Name.EndsWith("Handler") || 
                              t.Name.EndsWith("Service"))
                   .AsImplementedInterfaces()
                   .InstancePerLifetimeScope();
        }
    }
}
```

#### Presentation Module Example:
```csharp
// [MyProject.Presentation]/Modules/PresentationModule.cs
using Autofac;

namespace [MyProject.Presentation].Modules
{
    public class PresentationModule : Module
    {
        protected override void Load(ContainerBuilder builder)
        {
            // Register ViewModels
            builder.RegisterAssemblyTypes(typeof(PresentationModule).Assembly)
                   .Where(t => t.Name.EndsWith("ViewModel"))
                   .AsSelf()
                   .InstancePerDependency();
                   
            // Register Views if needed
            builder.RegisterAssemblyTypes(typeof(PresentationModule).Assembly)
                   .Where(t => t.Name.EndsWith("View") || 
                              t.Name.EndsWith("Window"))
                   .AsSelf()
                   .InstancePerDependency();
        }
    }
}
```

### **Container Bootstrap (Presentation Layer)**

#### App.xaml.cs or Program.cs:
```csharp
using Autofac;
using Autofac.Extensions.DependencyInjection;
using Microsoft.Extensions.DependencyInjection;
using System.Reflection;

public partial class App : Application
{
    public IContainer Container { get; private set; }
    
    protected override void OnStartup(StartupEventArgs e)
    {
        var builder = new ContainerBuilder();
        
        // Get all assemblies from current domain (avoids hardcoding)
        var assemblies = AppDomain.CurrentDomain.GetAssemblies()
            .Where(a => a.FullName?.StartsWith("[YourProjectNamespace]") == true)
            .ToArray();
            
        // Register all modules from assemblies using assembly scanning
        builder.RegisterAssemblyModules(assemblies);
        
        // Build container
        Container = builder.Build();
        
        base.OnStartup(e);
    }
}
```

### **Domain Interface → Infrastructure Implementation Pattern**

**Domain Layer (Interface):**
```csharp
// [MyProject.Domain]/Interfaces/IUserRepository.cs
namespace [MyProject.Domain].Interfaces
{
    public interface IUserRepository
    {
        Task<User> GetByIdAsync(int id);
        Task SaveAsync(User user);
    }
}
```

**Infrastructure Layer (Implementation):**
```csharp
// [MyProject.Infrastructure]/Repositories/UserRepository.cs
using [MyProject.Domain].Interfaces;

namespace [MyProject.Infrastructure].Repositories
{
    public class UserRepository : IUserRepository
    {
        public async Task<User> GetByIdAsync(int id)
        {
            // Implementation
        }
        
        public async Task SaveAsync(User user)
        {
            // Implementation
        }
    }
}
```

**Registration happens automatically** in `InfrastructureModule` via assembly scanning.

### **Module Registration Validation**
- [ ] Each layer has its own Module class
- [ ] Modules use assembly scanning (no hardcoded types)
- [ ] Presentation layer bootstraps container with assembly scanning
- [ ] Domain interfaces are implemented in Infrastructure layer
- [ ] All modules follow naming convention: `[ProjectName][LayerName]Module`

---

## 🎯 Layer-Specific Recreation Requirements

### 1. 📱 Presentation Layer `[MyProject.Presentation]`

**Project Type & Framework:**
- **SDK:** `Microsoft.NET.Sdk`
- **OutputType:** `WinExe` (Windows executable)
- **TargetFramework:** `.NET 9.0-windows`
- **WPF Enabled:** `<UseWPF>true</UseWPF>`
- **Nullable Reference Types:** Enabled
- **Implicit Usings:** Enabled
- **Platforms:** `AnyCPU`, `x64`

**Required NuGet Packages:**
- `Autofac` (latest 8.x)
- `Autofac.Extensions.DependencyInjection` (latest 10.x)
- `DevExpressMvvm` (latest)
- `MahApps.Metro` (latest 2.x)
- `Microsoft.Extensions.Logging` (latest 8.x)
- `Microsoft.Extensions.Logging.Abstractions` (latest 8.x)
- `NLog` (latest 6.x)
- `NLog.Extensions.Logging` (latest 6.x)
- `System.Reactive` (latest 6.x)

**Project References:**
- Reference to `[MyProject.Application]`
- Reference to `[MyProject.Infrastructure]`

**Content & Resources:**
- `NLog.config` (copied to output directory)
- `Views\WindowPreview.xaml` (WPF XAML page)

### 2. 🏢 Application Layer `[MyProject.Application]`

**Project Setup:**
- **TargetFramework:** `net9.0`
- **SDK:** `Microsoft.NET.Sdk`
- **ImplicitUsings:** `enable`
- **Nullable:** `enable`
- **Platforms:** `AnyCPU;x64`

**Required NuGet Packages:**
- `Autofac` (latest 8.x)
- `Autofac.Extensions.DependencyInjection` (latest 10.x)
- `NLog` (latest 6.x)
- `NLog.Extensions.Logging` (latest 6.x)

**Project References:**
- Reference to `[MyProject.Domain]`

### 3. 🔧 Infrastructure Layer `[MyProject.Infrastructure]`

**Project Metadata:**
- **SDK:** `Microsoft.NET.Sdk`
- **TargetFramework:** `.NET 9.0` (`net9.0`)
- **ImplicitUsings:** Enabled
- **Nullable:** Enabled
- **Platforms:** `AnyCPU` and `x64`

**Required NuGet Packages:**
- `Autofac` (latest 8.x)
- `Autofac.Extensions.DependencyInjection` (latest 10.x)
- `NLog` (latest 6.x)
- `NLog.Extensions.Logging` (latest 6.x)
- `System.Reactive` (latest 6.x)

**Project References:**
- Reference to `[MyProject.Domain]`

### 4. 🎯 Domain Layer `[MyProject.Domain]`

**Project Properties:**
- **TargetFramework:** `net9.0`
- **ImplicitUsings:** `enable`
- **Nullable:** `enable`
- **Platforms:** `AnyCPU;x64`

**Folders:**
- `Content` folder (included in project structure)

**Dependencies:**
- **NuGet Packages:** None (Domain layer should be dependency-free)
- **Project References:** None (Domain layer has no dependencies)

---

## 📋 Complete Validation Checklist

**AI Agent MUST confirm before code generation:**

### Solution Structure:
- [ ] Solution file exists in repository root
- [ ] All four expected projects present and named per conventions
- [ ] Target framework is `net9.0` for all projects (except Presentation uses `net9.0-windows`)

### Layer Dependencies:
- [ ] Presentation references Application and Infrastructure only
- [ ] Application references Domain only
- [ ] Infrastructure references Domain only
- [ ] Domain has no project references

### Project-Specific Validations:
- [ ] **Presentation:** WPF enabled (not WinForms), all UI packages installed, contains NLog.config
- [ ] **Application:** Autofac and NLog packages installed with correct versions
- [ ] **Infrastructure:** All required packages including System.Reactive
- [ ] **Domain:** No NuGet packages, contains Content folder

### Code Quality:
- [ ] All ViewModels inherit correct base class
- [ ] DI and logging patterns match requirements
- [ ] Nullable reference types enabled across all projects
- [ ] ImplicitUsings enabled across all projects

### Module Validation:
- [ ] Each project has its own Autofac Module class
- [ ] Modules use assembly scanning (no hardcoded assembly names)
- [ ] Module naming follows convention: `[ProjectName][LayerName]Module`
- [ ] Presentation layer bootstraps container with `RegisterAssemblyModules()`
- [ ] Infrastructure module registers domain interface implementations

---

## ⚠️ Failure Patterns & Diagnostics

**AI Agent MUST recognize and act on:**
- Missing solution or project files: _"Build stopped: Solution file is missing."_
- All four projects present: _"Build stopped: All four projects exist in the solution."_
- WinForms presentation layer detected: _"Build stopped: Invalid presentation layer. Expected WPF (.NET 9-windows with UseWPF), found WinForms."_
- Wrong target framework: _"Project does not target .NET 9"_
- Missing required packages: _"Required NuGet package [PackageName] not found"_
- Dependency rule violations: _"Domain layer cannot reference other projects"_
- Missing required modules: _"Required Autofac module [ModuleName] not found in [LayerName]"_
- Hardcoded assembly registration: _"Assembly scanning required, hardcoded assembly names detected"_

---

## 🚀 AI Agent Step-by-Step Execution

### Phase 1: Initial Validation
1. **Check solution file existence**
2. **Validate all four projects are present**
3. **Confirm project naming conventions match placeholder mapping**
4. **Verify target frameworks (.NET 9)**

### Phase 2: Dependency Validation
1. **Check project reference hierarchy matches DDD rules**
2. **Validate NuGet package installations and versions**
3. **Confirm no circular dependencies exist**
4. **Verify Domain layer isolation (no dependencies)**

### Phase 3: Structure Validation
1. **Confirm WPF configuration for Presentation layer**
2. **Validate Content folder exists in Domain**
3. **Check NLog.config presence in Presentation**
4. **Verify platform targets (AnyCPU, x64)**

### Phase 4: Code Generation Preparation
1. **Load all validation results**
2. **Apply DDD, DI, and logging patterns**
3. **Use established templates and conventions**
4. **Reference documentation for implementation details**

---

## 📚 Reference Documentation Structure

This onboarding guide consolidates information from:
- **Project Mapping:** Placeholder conventions for generic documentation
- **Layer Recreation Guides:** Specific requirements for each DDD layer
- **Architecture Validation:** DDD principles and dependency rules
- **Package Management:** Exact versions and configurations required

---

## 🎯 Success Criteria

**Solution is ready for code generation when:**
- All validation checklist items are confirmed ✅
- Project structure matches DDD layered architecture ✅  
- All dependencies are correctly installed and referenced ✅
- Platform and framework targets are consistent ✅
- Failure patterns are identified and resolved ✅

**AI Agent must not proceed with code generation until all success criteria are met.**

---

## 🔄 Continuous Validation

**During development, AI Agent must:**
- Re-validate structure before each significant change
- Maintain DDD layer separation principles
- Follow established naming conventions and patterns
- Use placeholder mapping for documentation consistency
- Apply logging and DI patterns consistently across layers

---

*This guide ensures systematic, validated solution recreation and maintains architectural integrity throughout the development process.*