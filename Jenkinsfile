pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Setup Python') {
            steps {
                bat '''
                    python --version
                    python -m venv .venv
                    .venv\\Scripts\\python.exe -m pip install --upgrade pip
                    .venv\\Scripts\\python.exe -m pip install -r requirements.txt
                '''
            }
        }

        stage('Prepare Dataset') {
            steps {
                bat '''
                    .venv\\Scripts\\python.exe download_data.py
                '''
            }
        }

        stage('Train Model') {
            steps {
                bat '''
                    .venv\\Scripts\\python.exe app\\train.py
                '''
            }
        }

        stage('Run Tests') {
            steps {
                bat '''
                    .venv\\Scripts\\python.exe -m pytest -v
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                    docker build -t aquaguard:latest .
                '''
            }
        }

        stage('Deploy AquaGuard') {
            steps {
                bat '''
                    docker rm -f aquaguard-app 2>NUL
                    docker run -d --name aquaguard-app --network aqua_project_default --restart unless-stopped -p 5001:5000 aquaguard:latest
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '''
                    set HEALTHY=0

                    for /L %%i in (1,1,15) do (
                        if !HEALTHY! EQU 0 (
                            docker exec aquaguard-app python -c "import urllib.request; urllib.request.urlopen('http://127.0.0.1:5000/health', timeout=3)" 2>NUL

                            if !ERRORLEVEL! EQU 0 (
                                echo AquaGuard deployment is healthy!
                                set HEALTHY=1
                            ) else (
                                timeout /t 2 /nobreak >NUL
                            )
                        )
                    )

                    if !HEALTHY! EQU 0 (
                        echo Deployment health check failed!
                        docker logs --tail 50 aquaguard-app
                        exit /b 1
                    )
                '''
            }
        }
    }
}
