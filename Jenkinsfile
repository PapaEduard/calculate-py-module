pipeline {
    agent any

    environment {
        VENV_DIR   = "${WORKSPACE}/venv"
        REPORT_DIR = "${WORKSPACE}/reports"
        DIST_DIR   = "${WORKSPACE}/dist"
    }

    options {
        skipDefaultCheckout(true)
        timestamps()
    }

    stages {
        stage('Checkout') {
            steps {
                deleteDir()
                checkout scm
            }
        }

        stage('Setup Python environment') {
            steps {
                sh """
                  python3 -m venv ${VENV_DIR}
                  . ${VENV_DIR}/bin/activate
                  pip install --upgrade pip setuptools wheel
                  pip install -r requirements.txt
                  pip install -e .
                """
            }
        }

        stage('Security Scan with Bandit') {
            steps {
                sh """
                  . ${VENV_DIR}/bin/activate
                  pip install bandit
                  mkdir -p ${REPORT_DIR}
                  bandit -r src -f json -o ${REPORT_DIR}/bandit.json
                """
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh """
                  . ${VENV_DIR}/bin/activate
                  pip install pytest
                  mkdir -p ${REPORT_DIR}
                  pytest --junitxml=${REPORT_DIR}/tests.xml
                """
            }
        }

        stage('Build Distributions') {
            steps {
                sh """
                  . ${VENV_DIR}/bin/activate
                  python setup.py sdist bdist_wheel
                  mkdir -p ${DIST_DIR}
                  mv dist/* ${DIST_DIR}/
                """
            }
        }

        stage('Package Module') {
            steps {
                sh """
                  cd ${DIST_DIR}
                  tar czf ${WORKSPACE}/calculator_module.tar.gz *
                """
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}
