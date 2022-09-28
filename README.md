#Airflow, MinIO e Postgres com DockerCompose
Estrutura containerizada de um ambiente de execução com Apache Airflow, MinIO e PostgreSQL

## Requisitos de sistema
- Linux (A distribuição que você preferir)
- Docker ( >= 20.10.18 )
- Docker Compose (>= 1.29.2)
 
Obs.: O projeto pode ser executado no Windows, desde que habilitado o WSL-2.

## Estrutura de diretórios
Os três diretórios dentro de airflow, não precisam ser necessariamente criados antes de executar o compose. O compose vai criá-los, se necessário. Porém, se clonar este repositório, a estrutura será replicada em sua máquina.
```
.
├── .env
├── airflow
│   ├── dags
│   ├── logs
│   └── plugins
├── docker-compose.yaml
└── minio
    └── data
```

## Limitando o consumo de memória dos containers
Para evitar o consumo desenfreado  de memória RAM pelos containers orquestrados pelo Docker Compose, é imprescindível definir no docker-compose.yaml os limites por container.  Se subir os containers sem definição de limites, vai ser praticamente impossível utilizar a máquina aonde os containers estiverem rodando.

O trecho de código abaixo, foi definido e personalizado para cada um dos serviços. Ajuste-os conforme sua necessidade, tendo em mente que o mínimo requerido pelo Airflow é 4GB de RAM.
Neste projeto, limitei apenas o consumo de RAM. Também é possível limitar o uso de processador. Para mais detalhes, consulte a documentação oficial do Docker Compose.
```yaml
airflow-webserver:
    <<: *airflow-common
    # command: webserver
    command:  bash -c "airflow webserver & airflow scheduler"
    ports:
      - 8080:8080    
    deploy:
      resources:
        limits:
            #  Indica o limite máximo de memória a ser utilizada por este container
            memory: 4GB
        reservations:
            memory: 2GB
```

##  Preparando o ambiente
Na raiz do projeto, execute:
```bash
$ echo -e "AIRFLOW_UID=$(id -u)" > .env
```
> Leia os comentários no arquivo docker-compose.yaml e defina as variáveis de ambiente no arquivo .env. Se não houver definições no .env, os containers serão criados com os valores defaults indicados no arquivo compose.

Inicie a configuração inicial do airflow e database, como a seguir:
```bash
$ docker-compose up airflow-init
```
Após algumas mensagens no terminal, se tudo correr bem, a instrução será finalizada, indicando o código de saída 0. 
Inicie agora os containers:
```bash
$ docker-compose up
```
Se nem um erro ocorrer, em instantes o seu terminal irá exibir várias mensagens de log, indicando o funcionamento do webserver e do scheduler.

Para finalizar os containers, pressione ctrl+c e aguarde.

Se desejar subir os containers como daemons, execute o compose da seguinte forma:
```bash
$ docker-compose up -d
```

## Acessando os serviços

### Airflow
O Airflow poderá ser acessado pelo endereço: http://localhost:8080

### MinIO
O MinIO estará disponível no endereço: http://localhost:9001

## Documentações oficiais:
 - [Apache Airflow com Docker Compose](https://airflow.apache.org/docs/apache-airflow/stable/howto/docker-compose/index.html);
 - [MinIO](https://min.io/docs/minio/linux/index.html);
 - [Compose file reference](https://docs.docker.com/compose/compose-file/compose-file-v3/).