<h1>Table of contents</h1>

- [Newman](#newman)
  - [Prerequisites](#prerequisites)
  - [Installation](#installation)
    - [Global Installation](#global-installation)
  - [Basic Usage](#basic-usage)
- [Reports](#reports)
- [Jenkins](#jenkins)
  - [Trello API - Postman CLI - freestyle](#trello-api---postman-cli---freestyle)
  - [Trello API - Postman CLI - Jenkinsfile](#trello-api---postman-cli---jenkinsfile)
  - [Trello API - Newman - freestyle](#trello-api---newman---freestyle)
  - [Trello API - Newman - Jenkinsfile](#trello-api---newman---jenkinsfile)
---
## Newman

### Prerequisites

- **Node.js**: Newman runs on Node.js. Download and **install the latest version** from [nodejs.org](https://nodejs.org/).
- **npm**: Comes bundled with Node.js and is the package manager used to install Newman.

### Installation

#### Global Installation
Install Newman globally to run the `newman` command from anywhere:

```bash
npm install -g newman
```

Verify the installation:

```bash
newman --version
```
```bash
6.2.1
```

### Basic Usage

Assuming you have:
- `my-collection.json`: Your exported Postman collection
- `my-environment.json` (optional): Your environment variables file

**Run a collection and view console output**

```bash
newman run my-collection.json \
  --environment my-environment.json
```

**Disable SSL certificate validation**

```bash
newman run my-collection.json \
  --environment my-environment.json \
  --insecure
```

**Generate an HTML report**

1. Install the HTML reporter:
   ```bash
   npm install -g newman-reporter-html
   ```
2. Run with HTML reporting:
   ```bash
   newman run my-collection.json \
     --environment my-environment.json \
     --reporters cli,html \
     --reporter-html-export report.html
   ```
   This creates a `report.html` file with a browsable report.

---
## Reports

[GitHub Actions](https://github.com/ovidiocbba/running-newman-with-github-actions)  
[Gitlab CI](https://gitlab.com/ovidiomiranda/running-newman-with-gitlab-ci)

## Jenkins
Execute Jenkins
```shell
docker run -p 8080:8080 -p 50000:50000 --restart=on-failure -v jenkins_home:/var/jenkins_home --env JAVA_OPTS="-Dfile.encoding=UTF8" vdespa/jenkins-postman
```
```shell
docker run -d --name jenkins-postman \
  -p 8080:8080 \
  -p 50000:50000 \
  --restart=unless-stopped \
  -v jenkins_home:/var/jenkins_home \
  -e JAVA_OPTS="-Dfile.encoding=UTF8" \
  vdespa/jenkins-postman
```

### Trello API - Postman CLI - freestyle  
Secret text   
```POSTMAN_API_KEY```  
Credentials  
```postman-api-key```

**Build Steps**  
Execute shell
``` 
postman --version
postman login --with-api-key $POSTMAN_API_KEY
postman collection run  3652386-691b3649-d2eb-4339-bb87-590ae99f6c8a
``` 
### Trello API - Postman CLI - Jenkinsfile
```jenkinsfile
pipeline {
  agent any
  
  environment{
      POSTMAN_API_KEY = credentials("postman-api-key")
  }

  stages {
   
    stage('Postman CLI Login') {
      steps {
        sh 'postman login --with-api-key $POSTMAN_API_KEY'
        }
    }

    stage('Running collection') {
      steps {
        sh 'postman collection run "3652386-691b3649-d2eb-4339-bb87-590ae99f6c8a"'
      }
    }
  }
}
``` 
------
### Trello API - Newman - freestyle
Secret text   
```POSTMAN_API_KEY```  
Credentials  
```postman-api-key```

**Build Steps**  
Execute shell
``` 
newman --version
newman run https://api.getpostman.com/collections/3652386-691b3649-d2eb-4339-bb87-590ae99f6c8a?apikey=$POSTMAN_API_KEY --reporters cli,htmlextra,junit --reporter-htmlextra-export newman/report.html --reporter-junit-export newman/report.xml
``` 
**Publish HTML reports**  
Index page[s]   
```newman/report.html```
Report title   
```Newman```  
**Publish JUnit test result report**  
Test report XMLs  
```newman/report.xml```  
### Trello API - Newman - Jenkinsfile
```jenkinsfile
 pipeline {
  agent any
  
  environment{
      POSTMAN_API_KEY = credentials("postman-api-key")
  }

  stages {
   
     stage('Running collection') {
      steps {
        sh 'newman --version'
        sh 'newman run https://api.getpostman.com/collections/3652386-691b3649-d2eb-4339-bb87-590ae99f6c8a?apikey=$POSTMAN_API_KEY --reporters cli,htmlextra,junit --reporter-htmlextra-export newman/report.html --reporter-junit-export newman/report.xml'
      }
    }
  }
  post {
        always {
            publishHTML target: [
                reportName: 'Newman',
                reportDir: 'newman',
                reportFiles: 'report.html',
                reportTitles: 'Postman API tests',
                keepAll: true,
                alwaysLinkToLastBuild: true,
                allowMissing: false
            ]
            junit 'newman/report.xml'
        }
    }
}
``` 
**Notes:**  
Dashboard > Manage Jenkins > Script Console   
``` 
System.setProperty("hudson.model.DirectoryBrowserSupport.CSP", "sandbox allow-scripts;")
``` 
**Html Publisher**  
Dashboard > Manage Jenkins > Plugins
Install Html Publisher  
