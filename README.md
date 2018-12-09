# Kafka para Iniciantes

Neste laboratório vamos exercitar os conceitos mais básicos do [Kafka](https://kafka.apache.org/).<br/>
Trata-se de aprendizado pela prática então deixe a preguiça de lado.<br/>

O primeiro passo ter uma instalação funcional de Kafka. Para simplificar, vamos utilizar o [material](https://github.com/confluentinc/cp-docker-images) disponibilizado pela [Confluent](https://www.confluent.io/) no GitHub.com sob a licença Apache 2.0.<br/>
Tal material foi desenvolvido para [docker](https://www.docker.com/), portanto é indispensável que você tenha o mesmo instalado em sua máquina.<br/>
Não se preocupe, se você nunca usou, não tem prática ou anda enferrujado, com uma leitura básica é possível fazer o básico, o que será mais que suficiente para este laboratório.<br/>

> **A Confluent** <br/>
> A Confluent é uma empresa americana cujos fundadores, [Jay Kreps](https://twitter.com/jaykreps), [Neha Narkhede](https://twitter.com/nehanarkhede) e [Jun Rao](https://twitter.com/junrao), originalmente no Linkedin desenharam e desenvolveram o Apache Kafka.<br/>
> No início de 2011 o código do Kafka foi aberto à comunidade de software via a [Apache Software Foundation](https://www.apache.org/). (fonte: [Wikipedia](https://en.wikipedia.org/wiki/Apache_Kafka)).<br/>
> Em 2014 o três fundaram a Confluent, que hoje abriga cerca de 70% dos commiters do código core do Kafka.<br/>
> Além disso, a Confluent desenvolve outras aplicações para o ecossistema Kafka, algumas [open source](https://github.com/confluentinc/), outras disponíveis no [Confluent Enterprise](https://www.confluent.io/product/confluent-enterprise/).

Legal! Vamos para tela preta!!!

## Passo 1: Instalar o Docker

Vá até https://www.docker.com/get-started e escolha o instalador para a sua plataforma (Windows ou Mac).
Já se você é rootz e usa Linux então aqui estão as instruções de acordo com a plataforma:

#### Redhat, Centos, Oracle Linux ou qualquer outro baseado em Redhat.
https://docs.docker.com/install/linux/docker-ce/centos/

ou

#### Ubuntu ou qualquer outra distribuição baseada em Debian
https://docs.docker.com/install/linux/docker-ce/ubuntu/

## Passo 2: Clonar o projeto da Confluent
> Se ainda não tem o Git instalado, aqui seguem as instruções de instalação:
> https://git-scm.com/book/en/v2/Getting-Started-Installing-Git

Abra um terminal, navegue até um diretório de sua preferência de onde deseja executar o lab e inicie a clonagem do projeto a partir do GitHub:
```
git clone https://github.com/confluentinc/cp-docker-images
```
## Passo 3: Iniciar o Kafka
#### E um Docker crash course também...

Agora navegue na árvore de diretórios a seguir:
```
cd cp-docker-images/examples/kafka-single-node/
```

Está na hora de iniciar o Kafka. Aqui a simplicidade do Docker entra em ação:
```
docker-compose up -d
```
É isso! Com um único comando você levantou duas instâncias de sistema operacional Linux, uma com um Zookeeper, outra com um Kafka, ambos standalone, isto é, não clusterizados.<br/>
Ao executar o comando pela primeira vez, levará alguns poucos minutos para o Docker baixar as imagens corretas do repositório. Nas próximas vezes o comando retornará quase que instantaneamente.<br/>
Você também poderia utilizar o mesmo comando omitindo o `-d`, essa instrução significa **detached**, ou seja, o terminal será liberado após a inicialização dos serviços. A ausência do `-d` indica, claro, o contrário.

A qualquer momento, quando conveniente, você pode interromper os serviços com o comando abaixo:
```
docker-compose stop
```

Caso deseje descartar o laboratório, basta executar o comando a seguir.<br/>
Atenção! Diferente de `docker-compose stop` esta ação descarta integralmente as instâncias envolvidas, portanto, é irreversível:
```
docker-compose down
```

Se quiser checar se os serviços estão de pé, execute o comando a seguir:
```
docker-compose ps
```
A saída será algo como:
```
barbosa-mbp:kafka-single-node infobarbosa$ docker-compose ps
            Name                         Command            State              Ports
------------------------------------------------------------------------------------------------
kafka-single-node_kafka_1       /etc/confluent/docker/run   Up      0.0.0.0:9092->9092/tcp
kafka-single-node_zookeeper_1   /etc/confluent/docker/run   Up      2181/tcp, 2888/tcp, 3888/tcp
```

Se quiser saber apenas os serviços, basta executar o mesmo comando acrescido de `--services`
```
docker-compose ps --services
```

Para entrar em qualquer instância o comando será assim:
```
docker exec -it kafka-single-node_kafka_1 /bin/bash
```

Perceba que **kafka-single-node_kafka_1** é o nome da instância (ou container) gerado pelo output do comando `docker-compose ps`. Já sacou, né?<br/>
Você pode aprender mais sobre o comando `exec` [aqui](https://docs.docker.com/engine/reference/commandline/exec/#parent-command).<br/>
O que esse comando faz é abrir um shell diretamente na instância pra que você possa interagir via linha de comando. Finja que está fazendo um `ssh` para a máquina. :)

Para sair do shell e voltar o host, o Docker não vai aceitar coisas como `Ctrl+D` (exit) ou `Ctrl+C` (interrupt signal). Se fizer desta forma, o Docker vai entender que quer interromper a instância e literalmente ele fará isso! <br/>

A saída do shell deve ser feita dessa forma: `Ctrl+P` + `Ctrl+Q` (É! Eu sei! Aceita que dói menos...).

Uma vez com um shell aberto na máquina do Kafka, você já pode checar algumas coisas interessantes.
##### Checando o serviço do Kafka:

```
ps aux | grep kafka
```
##### Checando se a porta do Kafka está respondendo:
```
nc -vz kafka 9092
```

O mesmo vale para o Zookeeper:
```
docker exec -it kafka-single-node_zookeeper_1 /bin/bash

ps aux | grep zookeeper

nc -vz zookeeper 2181
```

Agora quero chamar sua atenção para os outputs do comando `ps`:
O primeiro na máquina do Kafka:
```
root@93b8c1328493:/# ps aux | grep kafka
root         1  3.6  2.1 3188888 334344 ?      Ssl  23:22   0:14 java -Xmx1G -Xms1G -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true -Xloggc:/var/log/kafka/kafkaServer-gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dkafka.logs.dir=/var/log/kafka -Dlog4j.configuration=file:/etc/kafka/log4j.properties -cp /usr/bin/../share/java/kafka/*:/usr/bin/../share/java/confluent-support-metrics/*:/usr/share/java/confluent-support-metrics/* io.confluent.support.metrics.SupportedKafka /etc/kafka/kafka.properties
```
O segundo na máquina do Zookeeper:
```
root@c53f90385f83:/# ps aux | grep zookeeper
root         1  0.4  0.5 2543740 89052 ?       Ssl  Dec08   4:12 java -Xmx512M -Xms512M -server -XX:+UseG1GC -XX:MaxGCPauseMillis=20 -XX:InitiatingHeapOccupancyPercent=35 -XX:+ExplicitGCInvokesConcurrent -Djava.awt.headless=true -Xloggc:/var/log/kafka/zookeeper-gc.log -verbose:gc -XX:+PrintGCDetails -XX:+PrintGCDateStamps -XX:+PrintGCTimeStamps -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M -Dcom.sun.management.jmxremote=true -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false -Dkafka.logs.dir=/var/log/kafka -Dlog4j.configuration=file:/etc/kafka/log4j.properties -cp /usr/bin/../share/java/kafka/*:/usr/bin/../share/java/confluent-support-metrics/*:/usr/share/java/confluent-support-metrics/* org.apache.zookeeper.server.quorum.QuorumPeerMain /etc/kafka/zookeeper.properties
```

Vamos nos concentrar no final de cada output, `/etc/kafka/kafka.properties` e `/etc/kafka/zookeeper.properties`. <br/>
Esses dois arquivos contém os parâmetros necessários para a inicialização do Kafka e Zookeeper, respectivamente.<br/>
A especificação de parâmetros para o Kafka pode ser encontrada [aqui](https://kafka.apache.org/documentation/#brokerconfigs).
Já o Zookeeper tem suas configurações descritas [aqui](https://zookeeper.apache.org/doc/r3.4.9/zookeeperAdmin.html#sc_configuration).

A rigor você não precisa se preocupar com essas configurações agora porque vamos visitá-las com alguma frequência durante o curso. Porém, é importante saber desde já que essas configurações afetam diretamente o desempenho tanto do cluster Kafka quanto de sua aplicação, portanto, conhecê-las pode ser a chave de uma solução bem sucedida. **Conhecê-las**, não decorá-las! ;)

Estamos prontos para as próximas aulas!
