# Lab 1: Building and Packaging a Java Application with Maven

## Objective
Learn how to build, test, package, and run a Java application using Apache Maven.

## Tools Used
- Java JDK
- Apache Maven
- Git
- Visual Studio Code

## Steps

### 1. Verify Java and Maven
```bash
java -version
mvn -version
```

### 2. Clone the Repository
```bash
git clone https://github.com/Ibrahim-Adel15/CalculatorMavenApp.git
cd CalculatorMavenApp
```

### 3. Run Unit Tests
```bash
mvn test
```

### 4. Build the Application
```bash
mvn clean package
```

### 5. Run the Application
```bash
java -jar target/calculator.jar
```


## Expected Results
- Java and Maven are installed.
- The repository is cloned successfully.
- Unit tests pass.
- Maven builds the application successfully.
- A JAR artifact is generated inside the `target/` directory.
- The application runs successfully.
Check screenshots folder

