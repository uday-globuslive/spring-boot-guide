# Environment and IDE Setup for Spring Development

This guide will walk you through setting up a complete environment for Spring and Spring Boot development, including installing the JDK, build tools, and configuring IntelliJ IDEA for optimal Spring development.

## 1. Java Development Kit (JDK) Installation

Spring Framework 6 and Spring Boot 3 require Java 17 or higher. Here's how to install the JDK on different operating systems:

### 1.1 Windows Installation

1. **Download the JDK:**
   - Visit [Oracle JDK Downloads](https://www.oracle.com/java/technologies/downloads/) or [OpenJDK](https://jdk.java.net/)
   - Download the appropriate JDK 17 (or higher) installer for Windows (.exe or .msi)

2. **Run the installer:**
   - Double-click the downloaded file
   - Follow the installation wizard instructions
   - Note the installation directory (e.g., `C:\Program Files\Java\jdk-17`)

3. **Set Environment Variables:**
   - Right-click on 'This PC' or 'My Computer' and select 'Properties'
   - Click on 'Advanced system settings'
   - Click on 'Environment Variables'
   - Under System Variables, click 'New'
   - Add `JAVA_HOME` with the value of your JDK installation path (e.g., `C:\Program Files\Java\jdk-17`)
   - Find the 'Path' variable, click 'Edit'
   - Add `%JAVA_HOME%\bin` to the list of paths

4. **Verify Installation:**
   - Open Command Prompt
   - Type `java -version` and press Enter
   - You should see output indicating Java 17 (or your installed version)

### 1.2 macOS Installation

1. **Using Homebrew (Recommended):**
   ```bash
   # Install Homebrew if you don't have it
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   
   # Install OpenJDK 17
   brew install openjdk@17
   
   # Create a symbolic link to make it available system-wide
   sudo ln -sfn $(brew --prefix)/opt/openjdk@17/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-17.jdk
   ```

2. **Manual Installation:**
   - Download the macOS package from [Oracle JDK Downloads](https://www.oracle.com/java/technologies/downloads/) or [OpenJDK](https://jdk.java.net/)
   - Run the installer package (.pkg file)
   - Follow the installation instructions

3. **Verify Installation:**
   ```bash
   java -version
   ```

### 1.3 Linux Installation (Ubuntu/Debian)

1. **Install OpenJDK:**
   ```bash
   # Update package lists
   sudo apt update
   
   # Install OpenJDK 17
   sudo apt install openjdk-17-jdk
   ```

2. **Set Environment Variables:**
   ```bash
   # Add to ~/.bashrc or ~/.zshrc
   echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
   echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
   
   # Apply changes
   source ~/.bashrc
   ```

3. **Verify Installation:**
   ```bash
   java -version
   ```

### 1.4 Linux Installation (Red Hat/Fedora)

1. **Install OpenJDK:**
   ```bash
   # Update package lists
   sudo dnf update
   
   # Install OpenJDK 17
   sudo dnf install java-17-openjdk-devel
   ```

2. **Set Environment Variables:**
   ```bash
   # Add to ~/.bashrc
   echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk' >> ~/.bashrc
   echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
   
   # Apply changes
   source ~/.bashrc
   ```

3. **Verify Installation:**
   ```bash
   java -version
   ```

## 2. Build Tools Installation

### 2.1 Maven Installation

#### Windows

1. **Download Maven:**
   - Visit [Apache Maven Downloads](https://maven.apache.org/download.cgi)
   - Download the Binary zip archive (e.g., `apache-maven-3.9.4-bin.zip`)

2. **Extract the Archive:**
   - Extract to a directory of your choice (e.g., `C:\Program Files\Apache\maven`)

3. **Set Environment Variables:**
   - Add a new System Variable called `MAVEN_HOME` with the value of your Maven directory
   - Add `%MAVEN_HOME%\bin` to your Path variable

4. **Verify Installation:**
   ```cmd
   mvn --version
   ```

#### macOS

1. **Using Homebrew:**
   ```bash
   brew install maven
   ```

2. **Verify Installation:**
   ```bash
   mvn --version
   ```

#### Linux (Ubuntu/Debian)

1. **Install Maven:**
   ```bash
   sudo apt update
   sudo apt install maven
   ```

2. **Verify Installation:**
   ```bash
   mvn --version
   ```

### 2.2 Gradle Installation

#### Windows

1. **Download Gradle:**
   - Visit [Gradle Releases](https://gradle.org/releases/)
   - Download the binary-only zip file

2. **Extract the Archive:**
   - Extract to a directory (e.g., `C:\Gradle`)

3. **Set Environment Variables:**
   - Add a new System Variable called `GRADLE_HOME` with the value of your Gradle directory
   - Add `%GRADLE_HOME%\bin` to your Path variable

4. **Verify Installation:**
   ```cmd
   gradle --version
   ```

#### macOS

1. **Using Homebrew:**
   ```bash
   brew install gradle
   ```

2. **Verify Installation:**
   ```bash
   gradle --version
   ```

#### Linux (Ubuntu/Debian)

1. **Install Gradle:**
   ```bash
   sudo apt update
   sudo apt install gradle
   ```

2. **Verify Installation:**
   ```bash
   gradle --version
   ```

## 3. Installing and Configuring IntelliJ IDEA

### 3.1 Installation

1. **Download IntelliJ IDEA:**
   - Visit [JetBrains IntelliJ IDEA](https://www.jetbrains.com/idea/download/)
   - Choose Ultimate Edition (recommended for full Spring support) or Community Edition (free)
   - Download the appropriate installer for your operating system

2. **Run the Installer:**
   - Follow the installation wizard instructions
   - Select additional components as needed (e.g., 64-bit launcher, .java association)

3. **First Run Configuration:**
   - Launch IntelliJ IDEA
   - Choose to import settings or start with default settings
   - Sign in with your JetBrains account (if applicable)
   - Select your preferred UI theme

### 3.2 Installing Spring Boot Plugins

1. **Spring Assistant Plugin:**
   - Go to File > Settings > Plugins (or IntelliJ IDEA > Preferences > Plugins on macOS)
   - Select the "Marketplace" tab
   - Search for "Spring Assistant"
   - Click "Install" and restart when prompted

2. **Spring Boot Assistant:**
   - Similarly, install the "Spring Boot Assistant" plugin
   - Search in the Marketplace and install

3. **Spring Initializr and Assistant:**
   - Install "Spring Initializr and Assistant" for additional project creation options

### 3.3 Configuring IntelliJ IDEA for Spring Development

#### General Settings

1. **Optimize Import Settings:**
   - Go to File > Settings > Editor > General > Auto Import
   - Check "Optimize imports on the fly" and "Add unambiguous imports on the fly"

2. **Configure Annotation Processing:**
   - Go to File > Settings > Build, Execution, Deployment > Compiler > Annotation Processors
   - Check "Enable annotation processing"

3. **Set JDK and Maven/Gradle:**
   - Go to File > Settings > Build, Execution, Deployment > Build Tools > Maven (or Gradle)
   - Set "Maven home directory" to your Maven installation
   - Go to File > Project Structure > Project
   - Set the Project SDK to your installed JDK

#### Spring-Specific Settings

1. **Enable Spring Facet:**
   - For existing projects, right-click your project in the Project view
   - Select "Add Framework Support"
   - Check "Spring" and select the necessary Spring components

2. **Configure Spring Boot Run Configuration:**
   - Go to Run > Edit Configurations
   - Click the "+" button and select "Spring Boot"
   - Name your configuration (e.g., "MyApplication")
   - Set the Main class to your application's main class
   - Set any VM options or environment variables as needed

3. **Enable Actuator Endpoints Visualization:**
   - In a Spring Boot project, go to View > Tool Windows > Spring
   - Explore the "Beans", "Endpoints", and "Mappings" tabs

### 3.4 Creating a New Spring Boot Project with IntelliJ IDEA

1. **From IntelliJ IDEA Directly:**
   - Go to File > New > Project
   - Select "Spring Initializr" on the left
   - Configure:
     - Project SDK: Your installed JDK
     - Choose Maven or Gradle
     - Language: Java
     - Group and Artifact IDs
     - Spring Boot version
   - Click "Next" to select dependencies:
     - Core: Spring Boot DevTools, Lombok
     - Web: Spring Web
     - Data: Spring Data JPA, appropriate database driver
     - Others as needed
   - Click "Finish" to create the project

2. **From Spring Initializr Web:**
   - Visit [https://start.spring.io/](https://start.spring.io/)
   - Configure your project (same options as above)
   - Click "Generate" to download the project zip
   - In IntelliJ, go to File > Open and select the extracted project directory

### 3.5 Useful IntelliJ IDEA Features for Spring Development

1. **Spring Boot Run Dashboard:**
   - Visible at the bottom of the IDE when a Spring Boot project is open
   - Allows managing and monitoring Spring Boot applications

2. **Endpoints Tab:**
   - Shows all available Spring Boot Actuator endpoints
   - Provides a GUI to interact with them

3. **Bean Dependencies Diagram:**
   - Right-click on a Spring Bean
   - Select "Diagram" > "Show Dependencies"
   - Visualizes bean dependencies

4. **HTTP Client:**
   - Create .http files to test REST endpoints directly from the IDE
   - Example:
     ```
     ### Get all users
     GET http://localhost:8080/api/users
     
     ### Create a new user
     POST http://localhost:8080/api/users
     Content-Type: application/json
     
     {
       "name": "John Doe",
       "email": "john.doe@example.com"
     }
     ```
   - Click the "Run" icon next to each request to execute

5. **Database Tools:**
   - View > Tool Windows > Database
   - Connect to your project's database
   - Execute SQL, view tables, export/import data

6. **Live Templates for Spring:**
   - File > Settings > Editor > Live Templates
   - Several Spring-specific templates are included
   - Add your own by clicking the "+" button

## 4. Setting Up Docker (Optional but Recommended)

Docker is useful for running databases, message brokers, and other dependencies for Spring applications.

### 4.1 Installing Docker

#### Windows

1. **Prerequisites:**
   - Windows 10 64-bit: Pro, Enterprise, or Education (Build 16299 or later)
   - Enable Hyper-V and Containers Windows features

2. **Download and Install Docker Desktop:**
   - Visit [Docker Desktop for Windows](https://www.docker.com/products/docker-desktop)
   - Download the installer
   - Run the installer and follow the instructions

3. **Verify Installation:**
   ```cmd
   docker --version
   docker-compose --version
   ```

#### macOS

1. **Download and Install Docker Desktop:**
   - Visit [Docker Desktop for Mac](https://www.docker.com/products/docker-desktop)
   - Download the .dmg file
   - Open the .dmg file and drag Docker to the Applications folder

2. **Verify Installation:**
   ```bash
   docker --version
   docker-compose --version
   ```

#### Linux

1. **Install Docker Engine:**
   ```bash
   # Update package index
   sudo apt update
   
   # Install prerequisites
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   
   # Add Docker's official GPG key
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
   
   # Add the Docker repository
   sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
   
   # Update package index again
   sudo apt update
   
   # Install Docker
   sudo apt install docker-ce
   
   # Add current user to docker group (avoid using sudo for docker)
   sudo usermod -aG docker ${USER}
   ```

2. **Install Docker Compose:**
   ```bash
   sudo curl -L "https://github.com/docker/compose/releases/download/v2.18.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
   
   sudo chmod +x /usr/local/bin/docker-compose
   ```

3. **Verify Installation:**
   ```bash
   docker --version
   docker-compose --version
   ```

### 4.2 Setting Up Docker in IntelliJ IDEA

1. **Install Docker Integration Plugin:**
   - Go to File > Settings > Plugins
   - Search for "Docker" in the Marketplace
   - Install the "Docker" plugin and restart

2. **Configure Docker Connection:**
   - Go to File > Settings > Build, Execution, Deployment > Docker
   - Click "+" to add a new Docker configuration
   - Select connection type (typically "TCP socket" on Linux or "Docker for Windows/Mac")
   - Test the connection

3. **Using Docker Compose with IntelliJ:**
   - Create a `docker-compose.yml` file in your project
   - Example for a PostgreSQL database:
     ```yaml
     version: '3.8'
     services:
       postgres:
         image: postgres:14
         container_name: postgres_db
         ports:
           - "5432:5432"
         environment:
           POSTGRES_USER: postgres
           POSTGRES_PASSWORD: postgres
           POSTGRES_DB: mydatabase
         volumes:
           - postgres_data:/var/lib/postgresql/data
     
     volumes:
       postgres_data:
     ```
   - Right-click on the `docker-compose.yml` file
   - Select "Run" to start the containers

## 5. Version Control Setup

### 5.1 Git Installation

#### Windows

1. **Download Git:**
   - Visit [Git for Windows](https://gitforwindows.org/)
   - Download the installer

2. **Run the Installer:**
   - Select components (default is usually fine)
   - Choose the default editor (e.g., Visual Studio Code, Nano, Vim)
   - Configure the PATH environment
   - Configure line ending conversions

3. **Verify Installation:**
   ```cmd
   git --version
   ```

#### macOS

1. **Using Homebrew:**
   ```bash
   brew install git
   ```

2. **Verify Installation:**
   ```bash
   git --version
   ```

#### Linux

1. **Install Git:**
   ```bash
   sudo apt update
   sudo apt install git
   ```

2. **Verify Installation:**
   ```bash
   git --version
   ```

### 5.2 Git Configuration

1. **Set Global User Information:**
   ```bash
   git config --global user.name "Your Name"
   git config --global user.email "your.email@example.com"
   ```

2. **Set Default Branch Name:**
   ```bash
   git config --global init.defaultBranch main
   ```

3. **Configure Line Endings:**
   - For Windows:
     ```bash
     git config --global core.autocrlf true
     ```
   - For macOS/Linux:
     ```bash
     git config --global core.autocrlf input
     ```

### 5.3 Git Integration in IntelliJ IDEA

1. **Configure Git:**
   - Go to File > Settings > Version Control > Git
   - Set the path to the Git executable if not auto-detected

2. **Create a New Repository:**
   - VCS > Create Git Repository
   - Select your project directory

3. **Clone a Repository:**
   - File > New > Project from Version Control
   - Enter the repository URL

4. **Common Git Operations in IntelliJ:**
   - Commit: Ctrl+K (Cmd+K on Mac)
   - Push: Ctrl+Shift+K (Cmd+Shift+K on Mac)
   - Pull: Ctrl+T (Cmd+T on Mac)
   - View Log: Alt+9 (Cmd+9 on Mac)

## 6. Database Setup

### 6.1 Installing MySQL

#### Windows

1. **Download MySQL Installer:**
   - Visit [MySQL Downloads](https://dev.mysql.com/downloads/installer/)
   - Download the MySQL Installer

2. **Run the Installer:**
   - Choose "Developer Default" or "Custom" setup
   - Follow the installation wizard
   - Set the root password

#### macOS

1. **Using Homebrew:**
   ```bash
   brew install mysql
   ```

2. **Start MySQL Service:**
   ```bash
   brew services start mysql
   ```

3. **Secure MySQL Installation:**
   ```bash
   mysql_secure_installation
   ```

#### Linux

1. **Install MySQL:**
   ```bash
   sudo apt update
   sudo apt install mysql-server
   ```

2. **Start MySQL Service:**
   ```bash
   sudo systemctl start mysql
   sudo systemctl enable mysql
   ```

3. **Secure MySQL Installation:**
   ```bash
   sudo mysql_secure_installation
   ```

### 6.2 Installing PostgreSQL

#### Windows

1. **Download PostgreSQL Installer:**
   - Visit [PostgreSQL Downloads](https://www.postgresql.org/download/windows/)
   - Download the installer

2. **Run the Installer:**
   - Follow the installation wizard
   - Set the superuser (postgres) password
   - Select components to install (e.g., pgAdmin)

#### macOS

1. **Using Homebrew:**
   ```bash
   brew install postgresql
   ```

2. **Start PostgreSQL Service:**
   ```bash
   brew services start postgresql
   ```

#### Linux

1. **Install PostgreSQL:**
   ```bash
   sudo apt update
   sudo apt install postgresql postgresql-contrib
   ```

2. **Start PostgreSQL Service:**
   ```bash
   sudo systemctl start postgresql
   sudo systemctl enable postgresql
   ```

### 6.3 Database Integration in IntelliJ IDEA

1. **Connect to a Database:**
   - View > Tool Windows > Database
   - Click "+" and select the database type (MySQL, PostgreSQL, etc.)
   - Enter connection details (host, port, credentials)
   - Test the connection and click "OK"

2. **Database Explorer Features:**
   - Browse tables, views, and schemas
   - Execute SQL queries
   - Generate diagrams
   - Export and import data

3. **Database Console:**
   - Right-click a database connection
   - Select "Open Console"
   - Write and execute SQL queries with auto-completion

## 7. Additional Tools and Configurations

### 7.1 Postman for API Testing

1. **Download and Install Postman:**
   - Visit [Postman Downloads](https://www.postman.com/downloads/)
   - Download the appropriate installer for your OS
   - Run the installer

2. **Basic Usage for Testing Spring Boot APIs:**
   - Create a new collection for your project
   - Add requests for different endpoints
   - Set request methods (GET, POST, PUT, DELETE)
   - Add request bodies, headers, and authentication
   - Create environment variables for different environments

### 7.2 Lombok Configuration

Lombok is a Java library that reduces boilerplate code.

1. **Add Lombok to Your Project:**
   ```xml
   <!-- Maven -->
   <dependency>
       <groupId>org.projectlombok</groupId>
       <artifactId>lombok</artifactId>
       <version>1.18.26</version>
       <scope>provided</scope>
   </dependency>
   ```
   
   ```groovy
   // Gradle
   compileOnly 'org.projectlombok:lombok:1.18.26'
   annotationProcessor 'org.projectlombok:lombok:1.18.26'
   ```

2. **Install Lombok Plugin in IntelliJ IDEA:**
   - Go to File > Settings > Plugins
   - Search for "Lombok" in the Marketplace
   - Install the "Lombok" plugin and restart

3. **Enable Annotation Processing:**
   - Go to File > Settings > Build, Execution, Deployment > Compiler > Annotation Processors
   - Check "Enable annotation processing"

### 7.3 Code Quality Tools

1. **SonarLint Plugin:**
   - Go to File > Settings > Plugins
   - Search for "SonarLint" in the Marketplace
   - Install the "SonarLint" plugin and restart

2. **CheckStyle Plugin:**
   - Similarly, install the "CheckStyle-IDEA" plugin

3. **Google Java Format Plugin:**
   - Install the "google-java-format" plugin for automatic code formatting

### 7.4 Application Properties Templates

Create templates for common Spring Boot application properties:

1. **Development Properties Template:**
   ```properties
   # Application
   spring.application.name=my-application
   
   # Server
   server.port=8080
   
   # Database
   spring.datasource.url=jdbc:h2:mem:devdb
   spring.datasource.username=sa
   spring.datasource.password=
   spring.datasource.driver-class-name=org.h2.Driver
   
   # JPA/Hibernate
   spring.jpa.hibernate.ddl-auto=create-drop
   spring.jpa.show-sql=true
   spring.jpa.properties.hibernate.format_sql=true
   
   # H2 Console
   spring.h2.console.enabled=true
   spring.h2.console.path=/h2-console
   
   # Logging
   logging.level.org.springframework=INFO
   logging.level.com.yourpackage=DEBUG
   
   # Actuator
   management.endpoints.web.exposure.include=*
   management.endpoint.health.show-details=always
   ```

2. **Production Properties Template:**
   ```properties
   # Application
   spring.application.name=my-application
   
   # Server
   server.port=8080
   
   # Database
   spring.datasource.url=jdbc:mysql://localhost:3306/proddb
   spring.datasource.username=${DB_USERNAME}
   spring.datasource.password=${DB_PASSWORD}
   spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
   
   # JPA/Hibernate
   spring.jpa.hibernate.ddl-auto=none
   spring.jpa.show-sql=false
   
   # Logging
   logging.level.org.springframework=WARN
   logging.level.com.yourpackage=INFO
   
   # Actuator
   management.endpoints.web.exposure.include=health,info,metrics
   management.endpoint.health.show-details=never
   ```

## 8. Troubleshooting Common Setup Issues

### 8.1 JDK Issues

1. **Multiple JDK Versions:**
   - Ensure `JAVA_HOME` points to the correct JDK version
   - Check the Path variable for conflicting entries
   - Use `java -version` to verify the active JDK

2. **No JDK Found:**
   - Verify JDK installation path
   - Check environment variables
   - Reinstall JDK if necessary

### 8.2 Maven/Gradle Issues

1. **Dependency Resolution:**
   - Clear local repository:
     - For Maven: delete `~/.m2/repository` directory (or parts of it)
     - For Gradle: run `gradle --refresh-dependencies`
   
2. **Plugin Execution:**
   - Check the plugin version compatibility with your JDK
   - Update the build tool to the latest version

### 8.3 IntelliJ IDEA Issues

1. **Project Not Recognizing Spring:**
   - Ensure Spring facet is enabled
   - Reload Maven/Gradle project
   - Invalidate caches: File > Invalidate Caches / Restart

2. **Auto-completion Not Working:**
   - Check if annotation processing is enabled
   - Rebuild the project: Build > Rebuild Project
   - Verify required plugins are installed

3. **Run Configuration Issues:**
   - Delete and recreate run configurations
   - Check main class path
   - Verify working directory setting

### 8.4 Database Connection Issues

1. **Connection Refused:**
   - Verify database service is running
   - Check host, port, and credentials
   - Test with command-line client

2. **Authentication Failed:**
   - Reset database password if necessary
   - Check username case sensitivity
   - Verify database user permissions

## 9. Optimizing Development Workflow

### 9.1 IntelliJ IDEA Shortcuts for Spring Development

| Action | Windows/Linux | macOS |
|--------|--------------|-------|
| Generate code (constructors, getters, etc.) | Alt+Insert | Cmd+N |
| Reformat code | Ctrl+Alt+L | Cmd+Opt+L |
| Find usages | Alt+F7 | Opt+F7 |
| Go to declaration | Ctrl+B | Cmd+B |
| Go to implementation | Ctrl+Alt+B | Cmd+Opt+B |
| Show error description | Ctrl+F1 | Cmd+F1 |
| Quick fix | Alt+Enter | Opt+Enter |
| Run application | Shift+F10 | Ctrl+R |
| Debug application | Shift+F9 | Ctrl+D |
| Find in project | Ctrl+Shift+F | Cmd+Shift+F |
| Find and replace in project | Ctrl+Shift+R | Cmd+Shift+R |

### 9.2 Live Templates for Spring

Create custom live templates for common Spring patterns:

1. **Go to Settings:**
   - File > Settings > Editor > Live Templates

2. **Create a Spring Group:**
   - Click "+" and select "Template Group"
   - Name it "Spring"

3. **Add Template Examples:**

   - **Controller Method Template:**
     - Abbreviation: `getmapping`
     - Template text:
       ```java
       @GetMapping("/$path$")
       public ResponseEntity<$returnType$> $methodName$() {
           $END$
           return ResponseEntity.ok().build();
       }
       ```
     - Define variables:
       - path: Expression: `complete()`
       - returnType: Expression: `classNameComplete()`
       - methodName: Expression: `camelCase(methodName)`

   - **Service Method Template:**
     - Abbreviation: `svcmethod`
     - Template text:
       ```java
       @Override
       public $returnType$ $methodName$($paramType$ $paramName$) {
           $END$
       }
       ```

   - **Repository Query Method:**
     - Abbreviation: `findby`
     - Template text:
       ```java
       List<$entityType$> findBy$fieldName$($fieldType$ $paramName$);
       ```

### 9.3 External Tool Integrations

1. **Integrate Maven/Gradle Build Tool Window:**
   - View > Tool Windows > Maven/Gradle

2. **Terminal Integration:**
   - View > Tool Windows > Terminal
   - Runs in project root directory

3. **External Documentation:**
   - Add Spring documentation as external resource:
     - File > Settings > Tools > External Resources
     - Add URL pattern: `https://docs.spring.io/spring-framework/docs/current/reference/html/%s`

## 10. Final Checklist Before Starting Development

- [x] JDK installed and configured
- [x] Maven and/or Gradle installed
- [x] IntelliJ IDEA installed with necessary plugins
- [x] Git installed and configured
- [x] Database installed and accessible
- [x] Docker installed (if needed)
- [x] Project structure set up correctly
- [x] Build runs successfully
- [x] IDE recognizes Spring components
- [x] Basic Spring Boot application runs

**Next Steps:** Proceed to [Spring Core Fundamentals](01-spring-core-fundamentals.md) to begin your Spring journey.