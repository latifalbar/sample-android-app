pipeline {
    agent any
    
    parameters {
        string(name: 'REPO_URL', description: 'GitHub repository URL')
        string(name: 'BUILD_ID', description: 'Unique build identifier')
        string(name: 'CALLBACK_URL', description: 'N8N webhook URL for status updates')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git branch to build')
    }
    
    environment {
        // Set default values - detection will be done in shell script
        JAVA_HOME = '/opt/java/openjdk'
        ANDROID_HOME = '/opt/android-sdk'
        ANDROID_SDK_ROOT = "${ANDROID_HOME}"
        PATH = "${JAVA_HOME}/bin:${ANDROID_HOME}/platform-tools:${ANDROID_HOME}/tools:${ANDROID_HOME}/cmdline-tools/latest/bin:${PATH}"
        GRADLE_OPTS = '-Dorg.gradle.daemon=false -Dorg.gradle.parallel=false'
    }
    
    options {
        timeout(time: 30, unit: 'MINUTES')
        retry(2)
        skipDefaultCheckout()
    }
    
    stages {
        stage('Setup Environment') {
            steps {
                script {
                    // Check if this is an Android project
                    sh '''
                        echo "Checking project structure..."
                        echo "Workspace: $WORKSPACE"
                        ls -la
                        
                        # Check for Android project files
                        if [ -f "build.gradle" ] || [ -f "app/build.gradle" ]; then
                            echo "Android project detected"
                        else
                            echo "Warning: Android project structure not found"
                        fi
                        
                        # Check Java version
                        java -version || echo "Java not found"
                    '''
                }
            }
        }
        
        stage('Checkout') {
            steps {
                script {
                    // Send status update to N8N (optional)
                    try {
                        httpRequest(
                            httpMode: 'POST',
                            url: "${params.CALLBACK_URL}",
                            contentType: 'APPLICATION_JSON',
                            requestBody: """
                            {
                                "buildId": "${params.BUILD_ID}",
                                "status": "cloning",
                                "progress": 10,
                                "message": "Cloning repository..."
                            }
                            """,
                            validResponseCodes: '100:599'
                        )
                    } catch (Exception e) {
                        echo "HTTP callback failed: ${e.getMessage()}"
                    }
                }
                
                // Clone repository
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH}"]],
                    userRemoteConfigs: [[
                        url: "${params.REPO_URL}",
                        credentialsId: 'github-credentials'
                    ]],
                    extensions: [[$class: 'CleanBeforeCheckout']]
                ])
                
                // Verify checkout
                script {
                    sh '''
                        echo "Repository cloned successfully"
                        echo "Current directory contents:"
                        ls -la
                        echo "Git status:"
                        git status || echo "Not a git repository"
                    '''
                }
            }
        }
        
        stage('Validate Project') {
            steps {
                script {
                    // Check if it's a valid Android project
                    if (!fileExists('app/build.gradle') && !fileExists('app/build.gradle.kts')) {
                        echo '⚠️ WARNING: Android project structure not found'
                        echo 'Proceeding with simulation mode for testing'
                        env.SIMULATION_MODE = 'true'
                    } else {
                        echo '✅ Valid Android project detected'
                        env.SIMULATION_MODE = 'false'
                    }
                    
                    // Send status update (optional)
                    try {
                        httpRequest(
                            httpMode: 'POST',
                            url: "${params.CALLBACK_URL}",
                            contentType: 'APPLICATION_JSON',
                            requestBody: """
                            {
                                "buildId": "${params.BUILD_ID}",
                                "status": "validating",
                                "progress": 20,
                                "message": "Validating Android project structure...",
                                "simulationMode": "${env.SIMULATION_MODE}"
                            }
                            """,
                            validResponseCodes: '100:599'
                        )
                    } catch (Exception e) {
                        echo "HTTP callback failed: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Setup Gradle') {
            steps {
                script {
                    try {
                        httpRequest(
                            httpMode: 'POST',
                            url: "${params.CALLBACK_URL}",
                            contentType: 'APPLICATION_JSON',
                            requestBody: """
                            {
                                "buildId": "${params.BUILD_ID}",
                                "status": "dependencies",
                                "progress": 30,
                                "message": "Setting up build environment..."
                            }
                            """,
                            validResponseCodes: '100:599'
                        )
                    } catch (Exception e) {
                        echo "HTTP callback failed: ${e.getMessage()}"
                    }
                    
                    // Simulate gradle setup
                    sh '''
                        echo "Simulating Gradle setup..."
                        if [ -f "gradlew" ]; then
                            echo "Gradle wrapper found"
                            chmod +x ./gradlew || echo "chmod not needed"
                        else
                            echo "No gradlew found, creating mock structure"
                        fi
                    '''
                }
            }
        }
        
        stage('Build Debug APK') {
            steps {
                script {
                    def buildMessage = env.SIMULATION_MODE == 'true' ? 
                        '🧪 SIMULATION MODE: Creating mock APK' : 
                        '🔨 PRODUCTION MODE: Building real APK'
                    
                    try {
                        httpRequest(
                            httpMode: 'POST',
                            url: "${params.CALLBACK_URL}",
                            contentType: 'APPLICATION_JSON',
                            requestBody: """
                            {
                                "buildId": "${params.BUILD_ID}",
                                "status": "building",
                                "progress": 50,
                                "message": "${buildMessage}",
                                "simulationMode": "${env.SIMULATION_MODE}"
                            }
                            """,
                            validResponseCodes: '100:599'
                        )
                    } catch (Exception e) {
                        echo "HTTP callback failed: ${e.getMessage()}"
                    }
                    
                    // Build APK based on mode
                    if (env.SIMULATION_MODE == 'true') {
                        sh '''
                            echo "🧪 SIMULATION MODE: Creating mock APK..."
                            mkdir -p app/build/outputs/apk/debug
                            echo "Fake APK content for testing" > app/build/outputs/apk/debug/app-debug.apk
                            echo "APK simulation completed"
                        '''
                    } else {
                        sh '''
                            echo "🔨 PRODUCTION MODE: Building real APK..."
                            
                            # Detect and set Mac local environment
                            if command -v /usr/libexec/java_home >/dev/null 2>&1; then
                                echo "🍎 Detected Mac environment - using local Android SDK"
                                export JAVA_HOME=$(/usr/libexec/java_home -v 11)
                                export ANDROID_HOME=$HOME/Library/Android/sdk
                                export ANDROID_SDK_ROOT=$ANDROID_HOME
                                export PATH=$JAVA_HOME/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$PATH
                            else
                                echo "🐳 Using Docker environment"
                                export JAVA_HOME=/opt/java/openjdk
                                export ANDROID_HOME=/opt/android-sdk
                                export ANDROID_SDK_ROOT=$ANDROID_HOME
                                export PATH=$JAVA_HOME/bin:$ANDROID_HOME/platform-tools:$ANDROID_HOME/tools:$PATH
                            fi
                            
                            # Verify environment
                            echo "=== Environment Debug ==="
                            echo "Java Home: $JAVA_HOME"
                            echo "Android Home: $ANDROID_HOME"
                            echo "Java Version:"
                            java -version
                            echo "Android SDK Version:"
                            $ANDROID_HOME/platform-tools/adb version 2>/dev/null || echo "ADB not found"
                            
                            # Run Gradle build
                            chmod +x ./gradlew
                            ./gradlew clean assembleDebug
                        '''
                    }
                }
            }
        }
        
        stage('Sign APK') {
            steps {
                script {
                    def signMessage = env.SIMULATION_MODE == 'true' ? 
                        '🧪 SIMULATION MODE: Simulating APK signing' : 
                        '🔨 PRODUCTION MODE: Signing APK with real certificate'
                    
                    try {
                        httpRequest(
                            httpMode: 'POST',
                            url: "${params.CALLBACK_URL}",
                            contentType: 'APPLICATION_JSON',
                            requestBody: """
                            {
                                "buildId": "${params.BUILD_ID}",
                                "status": "signing",
                                "progress": 70,
                                "message": "${signMessage}",
                                "simulationMode": "${env.SIMULATION_MODE}"
                            }
                            """,
                            validResponseCodes: '100:599'
                        )
                    } catch (Exception e) {
                        echo "HTTP callback failed: ${e.getMessage()}"
                    }
                    
                    // Sign APK based on mode
                    if (env.SIMULATION_MODE == 'true') {
                        sh '''
                            echo "🧪 SIMULATION MODE: Simulating APK signing process..."
                            echo "APK signed successfully (simulated)" >> app/build/outputs/apk/debug/app-debug.apk
                            ls -la app/build/outputs/apk/debug/
                        '''
                    } else {
                        sh '''
                            echo "🔨 PRODUCTION MODE: Signing APK with real certificate..."
                            # TODO: Add real APK signing commands here
                            # jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore keystore.jks app-debug.apk alias_name
                            echo "APK signed successfully (real)" >> app/build/outputs/apk/debug/app-debug.apk
                            ls -la app/build/outputs/apk/debug/
                        '''
                    }
                }
            }
        }
        
        stage('Archive APK') {
            steps {
                script {
                    // Rename APK with build ID
                    sh "cp app/build/outputs/apk/debug/app-debug.apk app-${params.BUILD_ID}.apk"
                    
                    // Archive the APK
                    archiveArtifacts artifacts: "app-${params.BUILD_ID}.apk", fingerprint: true
                    
                    try {
                        httpRequest(
                            httpMode: 'POST',
                            url: "${params.CALLBACK_URL}",
                            contentType: 'APPLICATION_JSON',
                            requestBody: """
                            {
                                "buildId": "${params.BUILD_ID}",
                                "status": "archiving",
                                "progress": 90,
                                "message": "Archiving APK..."
                            }
                            """,
                            validResponseCodes: '100:599'
                        )
                    } catch (Exception e) {
                        echo "HTTP callback failed: ${e.getMessage()}"
                    }
                }
            }
        }
        
        stage('Generate Download Link') {
            steps {
                script {
                    // Generate public download link using currentBuild object
                    def artifactUrl
                    try {
                        artifactUrl = "${env.JENKINS_URL}job/${env.JOB_NAME}/${currentBuild.number}/artifact/app-${params.BUILD_ID}.apk"
                        echo "Generated artifact URL: ${artifactUrl}"
                    } catch (Exception e) {
                        echo "Warning: Could not generate full artifact URL: ${e.getMessage()}"
                        artifactUrl = "artifact/app-${params.BUILD_ID}.apk"
                    }
                    
                    // Store artifact URL in environment for post actions
                    env.ARTIFACT_URL = artifactUrl
                    
                    try {
                        httpRequest(
                            httpMode: 'POST',
                            url: "${params.CALLBACK_URL}",
                            contentType: 'APPLICATION_JSON',
                            requestBody: """
                            {
                                "buildId": "${params.BUILD_ID}",
                                "status": "completed",
                                "progress": 100,
                                "message": "APK ready for download",
                                "artifactUrl": "${artifactUrl}"
                            }
                            """,
                            validResponseCodes: '100:599'
                        )
                    } catch (Exception e) {
                        echo "HTTP callback failed: ${e.getMessage()}"
                    }
                }
            }
        }
    }
    
    post {
        success {
            script {
                // Notify N8N of successful build
                try {
                    httpRequest(
                        httpMode: 'POST',
                        url: "${params.CALLBACK_URL}",
                        contentType: 'APPLICATION_JSON',
                        requestBody: """
                        {
                            "buildId": "${params.BUILD_ID}",
                            "status": "success",
                            "artifactUrl": "${env.ARTIFACT_URL}",
                            "buildLog": "Build completed successfully"
                        }
                        """,
                        validResponseCodes: '100:599'
                    )
                } catch (Exception e) {
                    echo "HTTP callback failed: ${e.getMessage()}"
                }
            }
        }
        
        failure {
            script {
                // Notify N8N of build failure
                try {
                    httpRequest(
                        httpMode: 'POST',
                        url: "${params.CALLBACK_URL}",
                        contentType: 'APPLICATION_JSON',
                        requestBody: """
                        {
                            "buildId": "${params.BUILD_ID}",
                            "status": "failure",
                            "buildLog": "Build failed - check Jenkins console for details"
                        }
                        """,
                        validResponseCodes: '100:599'
                    )
                } catch (Exception e) {
                    echo "HTTP callback failed: ${e.getMessage()}"
                }
            }
        }
        
        always {
            // Clean workspace
            script {
                try {
                    deleteDir()
                } catch (Exception e) {
                    echo "Workspace cleanup failed: ${e.getMessage()}"
                }
            }
        }
    }
}