pipeline {
    agent any

    environment {
        COMPOSE_FILE = "docker-compose.yml"
        CONTAINER_NAME = "cloudcomputingg"   // nama service dari docker-compose
        MYSQL_CONTAINER = "mysql_web"
        DOCKER_CREDENTIALS = 'dockerhub-credentials'
        REPO_URL = "https://github.com/akanabot/cloudcomptw.git"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "🔄 Checkout source code dari repo..."
                git branch: 'main', url: "${REPO_URL}"

                echo "🛠 Konfigurasi Git safe directory..."
                bat 'git config --global --add safe.directory "%cd%"'
            }
        }

        stage('Build Docker Images') {
            steps {
                echo "🏗️ Build Docker images menggunakan docker-compose..."
                bat """
                docker-compose -f ${COMPOSE_FILE} build
                """
            }
        }

        stage('Run Docker Containers') {
            steps {
                echo "🚀 Jalankan ulang container Laravel dan MySQL..."
                bat """
                echo ==== HENTIKAN CONTAINER LAMA ====
                docker stop ${CONTAINER_NAME} || echo "${CONTAINER_NAME} tidak berjalan"
                docker rm ${CONTAINER_NAME} || echo "${CONTAINER_NAME} sudah dihapus"
                docker stop ${MYSQL_CONTAINER} || echo "${MYSQL_CONTAINER} tidak berjalan"
                docker rm ${MYSQL_CONTAINER} || echo "${MYSQL_CONTAINER} sudah dihapus"

                echo ==== JALANKAN ULANG DOCKER COMPOSE ====
                docker-compose -f ${COMPOSE_FILE} down || exit 0
                docker-compose -f ${COMPOSE_FILE} up -d

                echo ==== CEK CONTAINER YANG AKTIF ====
                docker ps
                """
            }
        }

        stage('Verify Laravel Running') {
            steps {
                echo "🔍 Verifikasi Laravel container berjalan dengan benar..."
                bat """
                echo ==== TUNGGU 20 DETIK SUPAYA CONTAINER SIAP ====
                ping 127.0.0.1 -n 20 >nul

                echo ==== CEK STATUS LARAVEL ====
                curl -I http://127.0.0.1:8000 || echo "⚠ Gagal akses Laravel di port 8000"

                echo ==== CEK ISI HALAMAN LARAVEL ====
                curl http://127.0.0.1:8000 || echo "⚠ Gagal ambil halaman Laravel"
                """
            }
        }
    }

    post {
        success {
            echo '✅ Laravel berhasil dijalankan via Docker Compose di port 8000!'
        }
        failure {
            echo '❌ Build gagal, periksa log Jenkins Console Output.'
        }
        always {
            echo '🧹 Membersihkan container sementara...'
            bat "docker-compose -f ${COMPOSE_FILE} down"
        }
    }
}
