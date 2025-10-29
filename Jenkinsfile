pipeline {
  agent {
    docker {
      image 'python:3.12-slim'
      args  '-u' // stdout sin buffer
    }
  }

  options {
    timestamps()
    ansiColor('xterm')
    buildDiscarder(logRotator(numToKeepStr: '15'))
  }

  environment {
    PIP_CACHE_DIR = "${WORKSPACE}/.pip-cache"
    VENV = "${WORKSPACE}/.venv"
    REPORTS_DIR = "${WORKSPACE}/reports"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
        sh 'python --version'
      }
    }

    stage('Setup') {
      steps {
        sh '''
          python -m venv "$VENV"
          . "$VENV/bin/activate"
          python -m pip install --upgrade pip wheel
          mkdir -p "$REPORTS_DIR" "$PIP_CACHE_DIR"

          # Si tenés requirements-dev.txt lo usa, si no instala lo básico.
          if [ -f requirements-dev.txt ]; then
            PIP_CACHE_DIR="$PIP_CACHE_DIR" pip install -r requirements-dev.txt
          else
            PIP_CACHE_DIR="$PIP_CACHE_DIR" pip install \
              black isort flake8 pytest pytest-cov pytest-html coverage
          fi
        '''
      }
    }

    stage('Linter') {
      steps {
        sh '''
          . "$VENV/bin/activate"
          echo ">>> Black (check)"
          black --check .
          echo ">>> isort (check)"
          isort --check-only .
          echo ">>> flake8"
          flake8 . | tee "$REPORTS_DIR/flake8.log"
        '''
      }
    }

    stage('Unit Tests') {
      steps {
        sh '''
          . "$VENV/bin/activate"
          pytest -q \
            --junitxml="$REPORTS_DIR/junit.xml" \
            --cov=. --cov-report=xml:"$REPORTS_DIR/coverage.xml" \
            --cov-report=term-missing \
            --html="$REPORTS_DIR/pytest-report.html" --self-contained-html
        '''
      }
    }

    stage('Quality Gates (opcionales)') {
      when {
        expression { return fileExists('bandit.yaml') || fileExists('pyproject.toml') }
      }
      steps {
        sh '''
          . "$VENV/bin/activate"
          # Ejemplo de seguridad estática (opcional si no tenés bandit):
          python -m pip install bandit || true
          bandit -q -r . -f txt -o "$REPORTS_DIR/bandit.txt" || true
        '''
      }
    }
  }

  post {
    always {
      // Publicar reportes
      junit testResults: 'reports/junit.xml', allowEmptyResults: true
      cobertura coberturaReportFile: 'reports/coverage.xml', onlyStable: false, failNoReports: false

      // Reporte HTML de pytest
      publishHTML(target: [
        reportDir: 'reports',
        reportFiles: 'pytest-report.html',
        reportName: 'PyTest Report',
        keepAll: true,
        alwaysLinkToLastBuild: true
      ])

      // Guardar logs de linters y bandit
      archiveArtifacts artifacts: 'reports/**/*', onlyIfSuccessful: false, allowEmptyArchive: true

      // Marcar build UNSTABLE si hay errores de flake8 (sin romper el build)
      script {
        if (fileExists('reports/flake8.log')) {
          def flakeTxt = readFile('reports/flake8.log')
          if (flakeTxt?.trim()) {
            currentBuild.result = currentBuild.result ?: 'UNSTABLE'
            echo "⚠️  flake8 encontró observaciones. Revisá reports/flake8.log"
          }
        }
      }
    }
    cleanup {
      cleanWs(deleteDirs: true, notFailBuild: true)
    }
  }
}
