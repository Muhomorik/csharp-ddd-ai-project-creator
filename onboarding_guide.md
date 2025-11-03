# 🤖 AI Agent OnBoarding & Execution Guide
> Comprehensive instructions for AI agents to validate, recreate, and work with Domain-Driven Design .NET 9 solutions using Autofac DI and layered architecture.

---

## 📁 Required Configuration Files

### .gitignore Configuration

**REQUIRED:** Create a `.gitignore` file in the repository root.

**Instructions:**
1. Use the official .NET gitignore template from GitHub: https://github.com/github/gitignore/blob/main/VisualStudio.gitignore
2. Ensure the template covers:
   - .NET build artifacts (bin/, obj/, *.dll, *.exe, *.pdb)
   - Visual Studio files (.vs/, *.suo, *.user, *.sln.docstates)
   - JetBrains products (ReSharper, Rider): .idea/, *.DotSettings.user, _ReSharper*/
   - NuGet packages and artifacts
   - Platform-specific files (Windows, macOS)

**Alternative:** Use `dotnet new gitignore` command to generate a standard .NET gitignore file

### .editorconfig Configuration

**REQUIRED:** Create a `.editorconfig` file in the repository root to enforce consistent coding standards.

**Instructions:**
1. Use the official .NET EditorConfig template from Microsoft: https://learn.microsoft.com/en-us/dotnet/fundamentals/code-analysis/code-style-rule-options
2. Alternatively, use `dotnet new editorconfig` command to generate a standard .NET EditorConfig file
3. Ensure the configuration includes:
   - **Core settings:** charset (utf-8), indent style (spaces), indent size (4 for C#, 2 for XML/JSON)
   - **C# conventions:** var preferences, expression-bodied members, pattern matching
   - **Formatting rules:** new line preferences, indentation, spacing, wrapping
   - **Naming conventions:** PascalCase for types, interfaces with "I" prefix
   - **.NET conventions:** using directives organization, this. qualifiers, type preferences
4. Set `root = true` at the top of the file to prevent searching parent directories

**Key Areas to Configure:**
- File-specific settings for `*.cs`, `*.csproj`, `*.json`, `*.yml`, `*.md`
- Code style rules with severity levels (warning, suggestion, silent)
- Naming rules for interfaces, types, methods, properties
- Formatting preferences for braces, spacing, and line breaks

### Configuration Files Validation Checklist

**AI Agent MUST confirm before code generation:**
- [ ] `.gitignore` file exists in repository root
- [ ] `.gitignore` includes .NET, Visual Studio, and JetBrains templates
- [ ] `.editorconfig` file exists in repository root
- [ ] `.editorconfig` enforces consistent C# coding standards
- [ ] Both files are committed to version control

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

## 🎨 Common XAML UI Patterns

### Converting Window to MetroWindow

**Scenario:** Update a standard WPF Window to use MahApps.Metro's MetroWindow for modern styling.

**Requirements:**
- MahApps.Metro NuGet package installed (version 2.x as specified in Presentation Layer requirements)
- DO NOT extend MetroWindow - use it directly as-is
- Follow MahApps.Metro quick start guide: https://mahapps.com/docs/guides/quick-start

**Implementation Steps:**

#### Step 1: Update App.xaml Resource Dictionaries

Add MahApps.Metro resource dictionaries to `App.xaml`. **All file names are Case Sensitive!**

```xaml
<Application x:Class="[MyProject.Presentation].App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:local="clr-namespace:[MyProject.Presentation]"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <!-- MahApps.Metro resource dictionaries. Make sure that all file names are Case Sensitive! -->
                <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Controls.xaml" />
                <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Fonts.xaml" />
                <!-- Theme setting -->
                <ResourceDictionary Source="pack://application:,,,/MahApps.Metro;component/Styles/Themes/Light.Blue.xaml" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

#### Step 2: Update MainWindow.xaml

**Add MahApps.Metro namespace:**

```xaml
xmlns:mah="clr-namespace:MahApps.Metro.Controls;assembly=MahApps.Metro"
```

or

```xaml
xmlns:mah="http://metro.mahapps.com/winfx/xaml/controls"
```

**Replace Window tags with MetroWindow tags:**

**Before:**
```xaml
<Window x:Class="[MyProject.Presentation].MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:[MyProject.Presentation]"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <!-- Your content -->
    </Grid>
</Window>
```

**After:**
```xaml
<mah:MetroWindow x:Class="[MyProject.Presentation].MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:d="http://schemas.microsoft.com/expression/blend/2008"
        xmlns:mc="http://schemas.openxmlformats.org/markup-compatibility/2006"
        xmlns:local="clr-namespace:[MyProject.Presentation]"
        xmlns:mah="clr-namespace:MahApps.Metro.Controls;assembly=MahApps.Metro"
        mc:Ignorable="d"
        Title="MainWindow" Height="450" Width="800">
    <Grid>
        <!-- Your content -->
    </Grid>
</mah:MetroWindow>
```

#### Step 3: Update MainWindow.xaml.cs Code-Behind

**Option 1 (Recommended):** Remove the base class (partial class will inherit from XAML):

```csharp
namespace [MyProject.Presentation]
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow
    {
        public MainWindow()
        {
            InitializeComponent();
        }
    }
}
```

**Option 2:** Explicitly inherit from MetroWindow:

```csharp
using MahApps.Metro.Controls;

namespace [MyProject.Presentation]
{
    /// <summary>
    /// Interaction logic for MainWindow.xaml
    /// </summary>
    public partial class MainWindow : MetroWindow
    {
        public MainWindow()
        {
            InitializeComponent();
        }
    }
}
```

**Key Points:**
- **DO NOT extend MetroWindow** - use it directly without creating derived classes
- All MahApps.Metro resource file names are **Case Sensitive**
- The MetroWindow provides modern window chrome, styled title bar, and Metro-themed appearance
- Existing window content remains unchanged
- MahApps.Metro package must be installed (already specified in Presentation Layer requirements)

**Validation:**
- [ ] App.xaml includes all three MahApps.Metro resource dictionaries
- [ ] Resource dictionary file names are Case Sensitive
- [ ] MainWindow.xaml uses `<mah:MetroWindow>` tags instead of `<Window>`
- [ ] MahApps.Metro namespace is declared in XAML
- [ ] Code-behind either removes base class or explicitly inherits from MetroWindow
- [ ] Application builds without errors
- [ ] Window displays with Metro styling when run

---

### Replacing TextBlock Content with Status Display

**Scenario:** Replace a TextBlock with a status display showing "READY" and current date/time.

**Requirements:**
- Large green "READY" text
- Current date/time displayed below
- No code-behind
- No data binding
- Static XAML only

**Implementation:**

Replace the existing TextBlock with a StackPanel containing two TextBlocks:

```xaml
<StackPanel Margin="16" VerticalAlignment="Center" HorizontalAlignment="Center">
    <TextBlock Text="READY"
               FontSize="72"
               FontWeight="Bold"
               Foreground="Green"
               HorizontalAlignment="Center"/>
    <TextBlock Text="2025-11-02 21:35"
               FontSize="16"
               HorizontalAlignment="Center"
               Margin="0,10,0,0"/>
</StackPanel>
```

**Key Points:**
- **StackPanel** allows vertical stacking of elements
- **First TextBlock:** "READY" text with large font (72pt), bold weight, green color
- **Second TextBlock:** Date/time in system format (hardcoded as static text)
- **Alignment:** Both centered horizontally, StackPanel centered in parent
- **Spacing:** 10px top margin on date/time TextBlock for separation

**Before:**
```xaml
<TextBlock Margin="16"
           TextWrapping="Wrap"
           FontSize="14">
    Lorem ipsum dolor sit amet...
</TextBlock>
```

**After:**
```xaml
<StackPanel Margin="16" VerticalAlignment="Center" HorizontalAlignment="Center">
    <TextBlock Text="READY"
               FontSize="72"
               FontWeight="Bold"
               Foreground="Green"
               HorizontalAlignment="Center"/>
    <TextBlock Text="2025-11-02 21:35"
               FontSize="16"
               HorizontalAlignment="Center"
               Margin="0,10,0,0"/>
</StackPanel>
```

**Validation:**
- [ ] StackPanel replaces single TextBlock
- [ ] "READY" text is large (72pt), bold, and green
- [ ] Date/time text is smaller (16pt) and positioned below
- [ ] No code-behind modifications required
- [ ] No data binding used (static text only)

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

---

## 🔌 WPF App Startup & Dependency Injection (Extended Guide)

> Adapted from di_guide.md and aligned with this guide’s conventions (DDD layers, Autofac, .NET 9, and placeholder mapping).

### 🧭 High-level Startup Flow (WPF)
1. Initialize logging early (from configuration or defaults).
2. Build the DI container and create an application-wide lifetime scope.
3. Create the main window instance.
4. Provide a window provider (or similar) with the main window reference before resolving any view model that needs it.
5. Resolve the main/root view model from DI (constructor injection for its dependencies).
6. Set DataContext and show the main window.
7. On application exit, dispose the lifetime scope and container.

### 🧪 Example App lifecycle (pseudo-C#)
```csharp
using Autofac;
using System;
using System.Linq;
using System.Windows;

public partial class App : Application
{
    private IContainer? _container;
    private ILifetimeScope? _appScope;

    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        // 1) Logging
        InitializeLogging(); // e.g., load NLog/Serilog config

        // 2) DI container
        var builder = new ContainerBuilder();

        // Option A: auto-discover modules from your solution assemblies (recommended in this guide)
        var assemblies = AppDomain.CurrentDomain.GetAssemblies()
            .Where(a => a.FullName?.StartsWith("[YourProjectNamespace]") == true)
            .ToArray();
        builder.RegisterAssemblyModules(assemblies);

        // Option B: explicit registrations if you prefer
        ConfigureServices(builder); // optional

        _container = builder.Build();
        _appScope = _container.BeginLifetimeScope();

        // 3) Create main window instance
        var mainWindow = new MainWindow();

        // 4) Provide window reference before resolving VMs
        var windowProvider = _appScope.Resolve<IWindowProvider>();
        if (windowProvider is IWindowProviderWithSettableMainWindow settable)
        {
            settable.MainWindow = mainWindow;
        }

        // 5) Resolve main view model
        var vm = _appScope.Resolve<IMainViewModel>();

        // 6) Bind and show
        mainWindow.DataContext = vm;
        mainWindow.Show();
    }

    protected override void OnExit(ExitEventArgs e)
    {
        _appScope?.Dispose();
        _container?.Dispose();
        base.OnExit(e);
    }
}
```

Notes:
- IWindowProviderWithSettableMainWindow is a generic placeholder; use any abstraction that allows setting and retrieving the main window reference without WPF coupling in lower layers.
- IMainViewModel stands in for your actual root VM interface.

### 🧩 Dependency registration recipe (Autofac)
Below are container registrations you can adapt. Prefer explicit modules per layer as specified earlier (InfrastructureModule, ApplicationModule, PresentationModule), but these patterns apply inside those modules as well.

#### 1) Logger injection (type-aware)
Automatically inject a logger tied to the component’s concrete type so classes can depend on ILogger without manual wiring.
```csharp
// Autofac example: supply a logger tied to the component's type
builder.RegisterCallback(cr =>
{
    cr.Registered += (sender, args) =>
    {
        args.ComponentRegistration.PipelineBuilding += (s, pb) =>
        {
            pb.Use(PipelinePhase.ParameterSelection, (ctx, next) =>
            {
                var implType = args.ComponentRegistration.Activator.LimitType;
                var newParams = ctx.Parameters.Union(new[]
                {
                    new ResolvedParameter(
                        (pi, c) => pi.ParameterType == typeof(ILogger),
                        (pi, c) => /* e.g. NLog */ LogManager.GetLogger(implType.FullName))
                });
                ctx.ChangeParameters(newParams);
                next(ctx);
            });
        };
    };
});
```
If you’re not using NLog, replace the GetLogger call with your logging library’s factory.

#### 2) ViewModels
- Register all classes ending with ViewModel as self (or interfaces), defaulting to transient (new instance per resolve).
- Ensure any constructor parameter of type IScheduler (Rx) receives DispatcherScheduler.Current so UI subscriptions marshal correctly.
```csharp
builder.RegisterAssemblyTypes(ThisAssembly)
    .Where(t => t.Name.EndsWith("ViewModel"))
    .AsSelf() // optionally .AsImplementedInterfaces()
    .WithParameter(new ResolvedParameter(
        (pi, c) => pi.ParameterType == typeof(IScheduler),
        (pi, c) => DispatcherScheduler.Current))
    .InstancePerDependency();

// Optional override for the root VM if it needs specific parameters
builder.RegisterType<MainViewModel>()
    .As<IMainViewModel>()
    .WithParameter(new TypedParameter(typeof(IScheduler), DispatcherScheduler.Current))
    .InstancePerDependency();
```

#### 3) Views and window provider
- Register WPF windows and controls as self.
- Provide a process-wide, single-instance window provider that holds references to important windows (set by App before resolving VMs).
```csharp
builder.RegisterType<MainWindow>().AsSelf();

builder.RegisterType<WindowProvider>()
    .As<IWindowProvider>()
    .SingleInstance();

public interface IWindowProvider { Window? MainWindow { get; } }
public interface IWindowProviderWithSettableMainWindow : IWindowProvider { Window? MainWindow { get; set; } }
```

#### 4) Application/services layer
- Register application services, domain services, repositories, and dispatchers.
- Prefer singletons for stateless services and in-memory repositories that represent runtime state; otherwise choose lifetimes per your needs.
```csharp
// Services
builder.RegisterType<ApplicationService>().As<IApplicationService>().SingleInstance();

// Domain services
builder.RegisterType<DomainService>().As<IDomainService>().SingleInstance();

// Repositories (runtime/in-memory)
builder.RegisterType<InMemoryRepository>().As<IRepository>().SingleInstance();

// Event dispatcher / message bus abstractions
builder.RegisterType<EventDispatcher>().As<IEventDispatcher>().SingleInstance();
```

#### 5) Utilities (imaging or other cross-cutting)
Group them under abstractions and register as singletons if stateless:
```csharp
builder.RegisterType<ImageConverter>().As<IImageConverter>().SingleInstance();
builder.RegisterType<ImageProcessingService>().As<IImageProcessingService>().SingleInstance();
```

#### 6) Reactive scheduling
- Register a background IScheduler for timers/intervals and inject DispatcherScheduler.Current for UI-bound work.
- Keep tests able to override schedulers.
```csharp
builder.RegisterInstance(TaskPoolScheduler.Default)
    .As<IScheduler>(); // background default for non-UI work

builder.RegisterType<PollingScheduler>()
    .As<IPollingScheduler>()
    .WithParameter(new TypedParameter(typeof(TimeSpan), TimeSpan.FromMilliseconds(50)))
    .WithParameter(new TypedParameter(typeof(int), /* concurrency or batch size */ 4))
    .SingleInstance();
```

#### 7) External process launcher or adapter
Abstract OS/process interaction behind an interface and register a singleton instance.
```csharp
builder.Register(ctx => new ProcessLauncher(/* path or settings from config */))
    .As<IProcessLauncher>()
    .SingleInstance();
```
Load the external path or options from configuration rather than hardcoding. Keep the adapter UI-agnostic.

#### 8) Final assembly scan (optional, safe defaults)
Do a last pass to auto-register remaining types without overriding explicit registrations.
```csharp
builder.RegisterAssemblyTypes(ThisAssembly)
    .AsSelf()
    .AsImplementedInterfaces()
    .PreserveExistingDefaults();
```

### 🧵 Threading and lifetime guidance
- UI thread marshalling: inject DispatcherScheduler.Current into any view model or service that updates WPF-bound state via Rx.
- Background work: inject a background IScheduler (e.g., TaskPoolScheduler.Default) for timers, intervals, or compute-bound observables.
- Scopes: WPF apps commonly use a single root application scope; long-lived services can be singletons. Transients are fine for VMs and short-lived operations.
- Disposal: ensure IDisposable services (including Rx subscriptions) are disposed on app exit. Prefer CompositeDisposable in VMs/services that subscribe to observables.

### 📝 Logging conventions (generic)
- Inject ILogger into every service/view model. Make it the first constructor parameter by convention.
- When logging exceptions, log the exception object directly. Avoid eager string formatting when your logging library supports deferred formatting.
- Resolve the logger from DI; do not create new logger instances manually inside methods.

### ✅ Minimal checklist when recreating App
- [ ] Logging initialized before any DI usage
- [ ] DI container built; application lifetime scope created
- [ ] Window provider registered as a singleton; MainWindow instance set before resolving VMs
- [ ] View models get DispatcherScheduler.Current; background services use a background IScheduler
- [ ] Services/repositories/utilities registered with appropriate lifetimes
- [ ] Presentation decides application lifetime; lower layers do not terminate the process
- [ ] Dispose the lifetime scope and container in OnExit

### 📦 Drop-in templates (rename types to yours)
- Root VM: IMainViewModel / MainViewModel
- Window provider: IWindowProvider + implementation with MainWindow property
- Background scheduler: IPollingScheduler (or similar) configured via TimeSpan + concurrency parameter
- Process adapter: IProcessLauncher with start/stop async methods
- Repositories/services: IRepository, IApplicationService, IDomainService

---
