def git_repo = 'https://github.com/ChaseRushville/bachelorrepo'
def branch = 'main'
def python_version = 'Python 3.10.7'

pipeline {
    agent {
        label 'agent1'
    }
    stages {
        stage('git checkout') {
            steps {
                timeout(time: 300, unit: 'SECONDS') {
                    echoBanner("STAGE: git checkout")
                    git url: git_repo,
                        branch: branch
                }
            }
        }
        stage('verify python version') {
            steps {
                timeout(time: 15, unit: 'SECONDS') {
                    echoBanner("STAGE: verify python version")
                    sh '[ "$(python3 --version)" == "' + python_version + '" ]'
                }
            }
        }
        stage('setup python venv') {
            steps {
                timeout(time: 120, unit: 'SECONDS') {
                    echoBanner("STAGE: setup python venv")
                    sh """
                    python3 -m virtualenv ${env.WORKSPACE}/.jenkinsvenv
                    source ${env.WORKSPACE}/.jenkinsvenv/bin/activate
                    python3 -m pip install --upgrade pip
                    python3 -m pip install --upgrade black
                    python3 -m pip install --upgrade flake8
                    python3 -m pip install --upgrade pytest
                    python3 -m pip install --upgrade sphinx
                    deactivate
                    """
                }
            }
        }
        stage('black') {
            steps {
                timeout(time: 30, unit: 'SECONDS') {
                    echoBanner("STAGE: black")
                    activateVenv()
                    sh """
                    black --extend-exclude="/\\.jenkinsvenv/ ."
                    """
                    deactivateVenv()
                }
            }
        }
        stage('flake8') {
            steps {
                timeout(time: 30, unit: 'SECONDS') {
                    echoBanner("STAGE: flake8")
                    activateVenv()
                    sh """
                    flake8 --extend-exclude=.jenkinsvenv .
                    """
                    deactivateVenv()
                }
            }
        }
        stage('pytest') {
            steps {
                timeout(time: 120, unit: 'SECONDS') {
                    echoBanner("STAGE: pytest")
                    activateVenv()
                    sh """
                    pytest $TEST_DIRECTORY
                    """
                    deactivateVenv()
                }
            }
        }
        stage('sphinx') {
            steps {
                timeout(time: 60, unit: 'SECONDS') {
                    echoBanner("STAGE: sphinx")
                    activateVenv()
                    sh """
                    sphinx -b html docs/source/ docs/build/
                    git add docs/build
                    git commit -m 'Build documentation via ${env.BUILD_TAG}'
                    git push
                    """
                    deactivateVenv()
                }
            }
        }
    }
    post {
        failure {
            echoBanner("Error during build!")
            echo currentBuild.rawBuild.getLog(10).join('\n')
        }
    }
}

def activateVenv() {
    sh "source ${env.WORKSPACE}/.jenkinsvenv/bin/activate"
}

def deactivateVenv() {
    sh 'deactivate'
}

def echoBanner(def msg) {
    echo """
        ===========================================

        ${msg}

        ===========================================
    """
}
