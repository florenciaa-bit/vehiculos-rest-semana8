pipeline {
    agent any

    environment {
        IMAGE_NAME = 'imagen_vehiculos'
        CONTAINER_NAME = 'contenedor_sucursal'
        APP_PORT = '9090'
        CONTAINER_PORT = '8080'
        APP_CONTEXT_PATH = 'vehiculosBuild'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                echo 'Obteniendo el código fuente desde GitHub...'
                checkout scm
            }
        }

        stage('Validate RDS MySQL Credentials') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'rds-jdbc-url',
                        variable: 'SPRING_DATASOURCE_URL'
                    ),
                    usernamePassword(
                        credentialsId: 'rds-mysql-credentials',
                        usernameVariable: 'SPRING_DATASOURCE_USERNAME',
                        passwordVariable: 'SPRING_DATASOURCE_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x

                        echo "Validando las credenciales de Amazon RDS MySQL."

                        case "$SPRING_DATASOURCE_URL" in
                          jdbc:mysql://*)
                            echo "La URL JDBC tiene el formato correcto."
                            ;;
                          *)
                            echo "ERROR: La URL debe comenzar con jdbc:mysql://"
                            exit 1
                            ;;
                        esac

                        echo "Las credenciales de RDS fueron cargadas correctamente."
                    '''
                }
            }
        }

        stage('Build WAR Artifact') {
            steps {
                sh '''
                    echo "Compilando el proyecto con Maven..."
                    mvn clean package -DskipTests

                    echo "Comprobando el archivo WAR generado..."
                    ls -lh target/vehiculosBuild.war
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    echo "Construyendo la imagen Docker..."
                    docker build -t "$IMAGE_NAME" .
                '''
            }
        }

        stage('Deploy Application Container') {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'rds-jdbc-url',
                        variable: 'SPRING_DATASOURCE_URL'
                    ),
                    usernamePassword(
                        credentialsId: 'rds-mysql-credentials',
                        usernameVariable: 'SPRING_DATASOURCE_USERNAME',
                        passwordVariable: 'SPRING_DATASOURCE_PASSWORD'
                    )
                ]) {
                    sh '''
                        set +x

                        echo "Eliminando el contenedor anterior si existe..."
                        docker stop "$CONTAINER_NAME" || true
                        docker rm "$CONTAINER_NAME" || true

                        echo "Desplegando la aplicación en Docker con Tomcat..."

                        docker run -d \
                          -p "$APP_PORT:$CONTAINER_PORT" \
                          --name "$CONTAINER_NAME" \
                          -e SPRING_DATASOURCE_URL="$SPRING_DATASOURCE_URL" \
                          -e SPRING_DATASOURCE_USERNAME="$SPRING_DATASOURCE_USERNAME" \
                          -e SPRING_DATASOURCE_PASSWORD="$SPRING_DATASOURCE_PASSWORD" \
                          "$IMAGE_NAME"
                    '''
                }
            }
        }

        stage('Inspect Container') {
            steps {
                sh '''
                    echo "Esperando el inicio de Tomcat y Spring Boot..."
                    sleep 25

                    echo "Estado del contenedor:"
                    docker ps --filter "name=$CONTAINER_NAME"

                    echo "Últimos registros del contenedor:"
                    docker logs --tail=80 "$CONTAINER_NAME"
                '''
            }
        }

        stage('Verify Swagger and API') {
            steps {
                sh '''
                    echo "Validando Swagger/OpenAPI..."
                    curl -f -s \
                      "http://localhost:$APP_PORT/$APP_CONTEXT_PATH/api-docs" \
                      || exit 1

                    echo "Validando el endpoint de vehículos..."
                    curl -f -s \
                      "http://localhost:$APP_PORT/$APP_CONTEXT_PATH/vehiculos" \
                      || exit 1

                    echo "La aplicación fue desplegada y validada correctamente."
                '''
            }
        }
    }
}
