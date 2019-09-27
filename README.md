# Welcome to Elastic stack (ELK) on Docker + Paloalto.

## ELK

Então, o que é o ELK Stack? "ELK" é a sigla para três projetos de código aberto: Elasticsearch, Logstash e Kibana. O Elasticsearch é um mecanismo de pesquisa e análise. O Logstash é um pipeline de processamento de dados do lado do servidor que ingere dados de várias fontes simultaneamente, os transforma e os envia para um "stash" como o Elasticsearch.
O Kibana permite que os usuários visualizem dados com tabelas e gráficos no Elasticsearch.

## Paloalto

- No Firewall é configurado para enviar 4 tipos de logs (trafego, ameaça, config e system) para porta 514 do ip do seu logstash, mas você pode escolher não enviar um desses logs nas do paloalto;

## Característica

- O mecanismo de pesquisa de código aberto, distribuído, RESTful, baseado em JSON. Fácil de usar, escalável e flexível, ganhou hiper-popularidade entre os usuários e uma empresa formada em torno dele, você sabe, para pesquisa.
- Um mecanismo de busca no coração, os usuários começaram a usar o Elasticsearch para logs e queriam facilmente ingeri-los e visualizá-los. Entre no Logstash, o poderoso pipeline de ingestão, e no Kibana, a ferramenta de visualização flexível.

## Instalação

Em docker Swarm execute:

```sh     
    $ git clone --branch PALOALTO_production https://gitlab.ifrr.edu.br/infra/elk.git
    $ cd elk
    $ docker stack deploy -c docker-stack.yml ELK
```

Após a instalação, adicione os arquivos .json ao elasticsearch:

```sh     
    $ cd kibana
    $ curl -XPUT http://<your-elasticsearch-server>:9200/_template/panos-traffic?pretty -H 'Content-Type: application/json' -d @traffic_template_mapping-v1.json
    $ curl -XPUT http://<your-elasticsearch-server>:9200/_template/panos-threat?pretty -H 'Content-Type: application/json' -d @threat_template_mapping-v1.json
```

Agora acesse o kibana <your-elasticsearch-server>:5601/app/kibana#/management/kibana/objects?_g=(refreshInterval:(pause:!f,value:5000),time:(from:now-15m,to:now)) e importe os arquivos de objetos salvos (nesta ordem):

- searches-base.json
- visualisations-base.json
- dashboards-base.json

## LifeCycle (Ciclo de vida)

As APIs de gerenciamento de ciclo de vida do índice (ILM) permitem automatizar como você deseja gerenciar seus índices ao longo do tempo. Em vez de simplesmente executar ações de gerenciamento em seus índices em um cronograma definido, você pode basear as ações em outros fatores, como tamanho do fragmento e requisitos de desempenho.

Configuração

no console do kibana (<your-elasticsearch-server>:5601/app/kibana#/dev_tools/console?_g=(refreshInterval:(pause:!f,value:5000),time:(from:now-15m,to:now)), crie 4 templates que faz a ligação do seu index ao life cicle policy:

```
PUT _template/ifrr_traffic_template
{
  "index_patterns": ["ifrr-panos-trafic*"],       
  "settings": {
    "number_of_shards": 1,
    "refresh_interval" : "60s",
    "number_of_replicas": 1,
    "index.lifecycle.name": "ifrr-traffic-policy",    
    "index.lifecycle.rollover_alias": "ifrr-panos-traffic"    
  }
}

PUT _template/ifrr_threat_template
{
  "index_patterns": ["ifrr-panos-threat*"],       
  "settings": {
    "number_of_shards": 1,
    "refresh_interval" : "60s",
    "number_of_replicas": 1,
    "index.lifecycle.name": "ifrr-threat-policy",    
    "index.lifecycle.rollover_alias": "ifrr-panos-threat"    
  }
}

PUT _template/ifrr_config_template
{
  "index_patterns": ["ifrr-panos-config*"],       
  "settings": {
    "number_of_shards": 1,
    "refresh_interval" : "60s",
    "number_of_replicas": 1,
    "index.lifecycle.name": "ifrr-config-policy",    
    "index.lifecycle.rollover_alias": "ifrr-panos-config"    
  }
}

PUT _template/ifrr_system_template
{
  "index_patterns": ["ifrr-panos-system*"],       
  "settings": {
    "number_of_shards": 1,
    "refresh_interval" : "60s",
    "number_of_replicas": 1,
    "index.lifecycle.name": "ifrr-system-policy",    
    "index.lifecycle.rollover_alias": "ifrr-panos-system"    
  }
}
```

Agora Crie os index para receber os logs do logstash:

```
PUT /%3Cifrr-panos-trafic-%7Bnow%2Fd%7D-000001%3E 
{
  "aliases": {
    "ifrr-panos-traffic": {}
  }
}

PUT /%3Cifrr-panos-threat-%7Bnow%2Fd%7D-000001%3E 
{
  "aliases": {
    "ifrr-panos-threat": {}
  }
}

PUT /%3Cifrr-panos-config-%7Bnow%2Fd%7D-000001%3E 
{
  "aliases": {
    "ifrr-panos-config": {}
  }
}

PUT /%3Cifrr-panos-system-%7Bnow%2Fd%7D-000001%3E 
{
  "aliases": {
    "ifrr-panos-system": {}
  }
}
```
