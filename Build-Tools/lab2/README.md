# Lab 2: Building and Packaging a Java Application with Gradle

## Objective

Build, test, package, and run a Java calculator application using Gradle as a build automation tool.

## Tools Used

- Java JDK 17
- Gradle 8.14.6
- Git
- Visual Studio Code
- Windows PowerShell

## Repository

[CalculatorGradleApp](https://github.com/Ibrahim-Adel15/CalculatorGradleApp)

## Implementation Steps

### 1. Verify Java and Gradle Installation

```bash
java -version
gradle -v
```

**Installed Versions:**

- Java: OpenJDK 17.0.18
- Gradle: 8.14.6

### 2. Clone the Repository

```bash
git clone https://github.com/Ibrahim-Adel15/CalculatorGradleApp.git
cd CalculatorGradleApp
```

### 3. Run Unit Tests

```bash
gradle test
```

**Result:**

```text
BUILD SUCCESSFUL in 15s
3 actionable tasks: 3 executed
```

### 4. Build and Package the Application

```bash
gradle clean build
```

**Result:**

```text
BUILD SUCCESSFUL in 2s
8 actionable tasks: 8 executed
```

### 5. Verify the Generated Artifact

```powershell
Get-ChildItem build\libs
```

**Generated Artifact:**

```text
build/libs/calculator.jar
```

Artifact size: 2,297 bytes.

### 6. Run the Application

```bash
java -jar build/libs/calculator.jar
```

The application was executed using the generated JAR file.
