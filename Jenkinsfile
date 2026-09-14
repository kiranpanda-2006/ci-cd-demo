stage('Deploy') {

    when {
        branch 'master'
    }

    steps {

        sh '''
            set -e

            DEPLOY_ROOT=/opt/cicd-demo/spring
            RELEASE_DIR="$DEPLOY_ROOT/releases/$BUILD_NUMBER"

            mkdir -p "$RELEASE_DIR"

            JAR_FILE=$(find target \
                -maxdepth 1 \
                -type f \
                -name '*.jar' \
                ! -name '*-sources.jar' \
                | head -1)

            if [ -z "$JAR_FILE" ]; then
                echo "ERROR: No JAR file found"
                exit 1
            fi

            echo "Deploying: $JAR_FILE"

            cp "$JAR_FILE" "$RELEASE_DIR/app.jar"

            if [ -f "$DEPLOY_ROOT/app.pid" ]; then

                OLD_PID=$(cat "$DEPLOY_ROOT/app.pid" || true)

                if [ -n "$OLD_PID" ] && \
                   kill -0 "$OLD_PID" 2>/dev/null; then

                    echo "Stopping old application: $OLD_PID"
                    kill "$OLD_PID" || true
                    sleep 3
                fi
            fi

            echo "Starting application..."

            JENKINS_NODE_COOKIE=dontKillMe \
            APP_VERSION="$BUILD_NUMBER" \
            nohup java \
                -jar "$RELEASE_DIR/app.jar" \
                > "$DEPLOY_ROOT/app.log" 2>&1 &

            echo $! > "$DEPLOY_ROOT/app.pid"

            echo "Application PID: $(cat "$DEPLOY_ROOT/app.pid")"
        '''
    }
}

stage('Smoke Test') {

    when {
        branch 'master'
    }

    steps {

        sh '''
            echo "Waiting for Spring Boot application..."

            for i in $(seq 1 30); do

                if curl -fs http://127.0.0.1:8081/api/health; then
                    echo ""
                    echo "================================="
                    echo "APPLICATION IS UP!"
                    echo "================================="
                    exit 0
                fi

                echo "Application not ready yet... attempt $i/30"
                sleep 2
            done

            echo "ERROR: Application failed to start"

            echo "===== APPLICATION LOG ====="
            cat /opt/cicd-demo/spring/app.log || true

            exit 1
        '''
    }
}}