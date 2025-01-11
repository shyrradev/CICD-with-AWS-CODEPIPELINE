### A Deep Dive into My AWS CodePipeline CICD Project

Some projects challenge you to step outside your comfort zone, teaching you not just technical skills but also resilience, problem-solving, and the beauty of integration. My recent project—building a robust CI/CD pipeline using AWS CodePipeline—was exactly that. It’s a story of technical ingenuity and personal growth, and I’m thrilled to share the details with you.

This project used AWS services like CodePipeline, Bitbucket, CodeBuild, Elastic Beanstalk, and RDS MySQL Database. The seamless integration between these services and the pay-as-you-go model of AWS made this an enlightening experience. But it wasn’t without its challenges, and overcoming them made the process even more rewarding.

Let’s dive into the details step by step.

---

### **Why AWS CodePipeline?**

Before jumping into the project, I want to explain why I chose AWS CodePipeline. One of the standout features is its ability to integrate effortlessly with other AWS services. Unlike traditional CI/CD tools like Jenkins, which require a dedicated, always-on server (increasing costs), CodePipeline is event-driven. It only uses compute resources when triggered, perfectly embodying AWS’s pay-for-what-you-use philosophy.

With this foundation set, I was ready to build a pipeline that automated the deployment of an application. My goal was to integrate the tech stack—including Bitbucket for version control, CodeBuild for building and testing, Elastic Beanstalk for hosting, and RDS MySQL for the database—into one cohesive pipeline.

---

### **Step 1: Configuring Elastic Beanstalk**

Elastic Beanstalk serves as the backbone of the application. It handles auto-scaling, load balancing, and instance management. Here’s how I configured it:

1. **Environment Setup**:
    
    - I created an Elastic Beanstalk environment configured for a Java application.
    - This environment included two t2.micro EC2 instances to start, with auto-scaling enabled for up to four instances.
2. **Load Balancer Configuration**:
    
    - Elastic Beanstalk automatically created a load balancer, which was linked to the EC2 instances.
    - This load balancer ensures high availability and distributes incoming traffic efficiently.
3. **Security Groups**:
    
    - Configured the security group to restrict access and allow only necessary inbound traffic.
    - The security group permitted traffic on port 80 for HTTP and port 443 for HTTPS.
4. **Environment Variables**:
    
    - I set up environment variables within Elastic Beanstalk to store sensitive information like database credentials securely.

With the environment ready, I could focus on setting up the database.

---

### **Step 2: Setting Up the RDS MySQL Database**

A reliable database is critical for any application. AWS RDS MySQL provided the managed service I needed. Here’s what I did:

1. **Creating the Database**:
    
    - I launched an RDS MySQL instance and configured it with the necessary settings, including multi-AZ deployment for high availability.
    - Created a database named "accounts" to store application data.
2. **Configuring Security Groups**:
    
    - I updated the RDS security group to allow inbound traffic from the Elastic Beanstalk environment on port 3306 (the default MySQL port).
3. **Verifying the Connection**:
    
    - SSH’d into one of the EC2 instances created by Elastic Beanstalk.
        
    - Installed the MySQL client (MariaDB) on the instance:
        
        ```bash
        sudo yum install mariadb -y
        ```
        
    - Connected to the RDS database using the endpoint provided by AWS:
        
        ```bash
        mysql -h <rds-endpoint> -u <username> -p
        ```
        
4. **Importing Database Backup**:
    
    - Downloaded the database backup from GitHub:
        
        ```bash
        wget <link-to-db-backup.sql>
        ```
        
    - Imported the backup into the RDS database:
        
        ```bash
        mysql -h <endpoint> -u <username> accounts < db_backup.sql -p
        ```
        

With the database set up and populated, I moved to the next phase—code migration.

---

### **Step 3: Migrating Code from GitHub to Bitbucket**

Bitbucket’s integration capabilities and compliance features made it a natural choice for this project. Here’s how I migrated the code:

1. **Setting Up a Bitbucket Repository**:
    
    - Created a free repository on Bitbucket.
        
    - Configured SSH keys for secure communication using:
        
        ```bash
        ssh-keygen -t rsa -b 2048 -C "email@example.com"
        ```
        
    - Added the public key to Bitbucket and tested the connection:
        
        ```bash
        ssh -T git@bitbucket.org
        ```
        
2. **Migrating the Code**:
    
    - Cloned the GitHub repository locally:
        
        ```bash
        git clone <github-repo-url>
        ```
        
    - Removed the old origin and added the Bitbucket repository as the new origin:
        
        ```bash
        git remote rm origin
        git remote add origin <bitbucket-ssh-url>
        git push -u origin master
        ```
        

The code was now securely hosted on Bitbucket, ready for integration with CodePipeline.

---

### **Step 4: Building with AWS CodeBuild**

CodeBuild eliminates the need for maintaining build servers, providing a fully managed build service. Here’s how I set it up:

1. **Configuring CodeBuild**:
    
    - Created a build project in CodeBuild and linked it to the Bitbucket repository.
        
    - Used the following `buildspec.yml` to define the build process:
        
        ```yaml
        version: 0.2
        phases:
          install:
            runtime-versions:
              java: corretto17
            commands:
              - apt-get update
              - apt-get install -y jq
          pre_build:
            commands:
              - echo "Pre-build phase: updating application properties"
              - sed -i "s/<DB_ENDPOINT>/<actual-db-endpoint>/g" src/main/resources/application.properties
          build:
            commands:
              - echo "Building the application"
              - mvn clean package
          post_build:
            commands:
              - echo "Build completed"
        artifacts:
          files:
            - target/*.jar
        ```
        
2. **Testing the Build**:
    
    - Triggered a test build to ensure everything worked as expected. The build completed successfully, creating the required artifact.

---

### **Step 5: Setting Up AWS CodePipeline**

CodePipeline orchestrates the entire CI/CD process. Here’s how I set it up:

1. **Source Stage**:
    
    - Integrated Bitbucket as the source repository using OAuth.
    - Configured the pipeline to trigger on every push to the `master` branch.
2. **Build Stage**:
    
    - Connected the pipeline to the CodeBuild project created in Step 4.
3. **Deploy Stage**:
    
    - Configured Elastic Beanstalk as the deployment target.

Once the pipeline was saved, I tested it by pushing a commit to Bitbucket. The pipeline triggered automatically, validating the entire process.

---

### **Step 6: Reflecting on the Journey**

After completing the project, I took a moment to reflect. Each step taught me something valuable, whether it was troubleshooting security group issues or fine-tuning the buildspec file. The automation worked flawlessly, streamlining the deployment process.

---

### **Challenges Faced**

1. **Security Group Configuration**:
    
    - Initially, misconfigured security groups prevented the EC2 instances from accessing the RDS database.
2. **Application Properties**:
    
    - Updating sensitive information in the application properties file required extra care to avoid hardcoding credentials.
3. **Learning Curve**:
    
    - Understanding the nuances of each AWS service took time, especially when integrating them.

---

### **Lessons Learned**

- **Efficiency Through Automation**: Automating deployments saves time and reduces errors.
- **Power of Integration**: AWS services work seamlessly together, making complex workflows easier to manage.
- **Continuous Learning**: Each challenge is an opportunity to grow.

---

### **Conclusion**

This project was more than just a technical exercise; it was a journey of growth and discovery. From configuring Elastic Beanstalk to integrating Bitbucket, every step reinforced the value of perseverance and adaptability.

To those embarking on similar journeys, embrace the challenges. They’re the best teachers. Share your thoughts or questions in the comments below—let’s learn and grow together!

---

**Connect with me on LinkedIn**: [Shayan Nazar](https://www.linkedin.com/in/shayan-nazar-068703209/)

What’s your biggest takeaway from this journey? Let’s discuss in the comments!
