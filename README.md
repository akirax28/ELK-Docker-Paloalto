# Welcome to Elastic stack (ELK) on Docker + Paloalto.

## ELK

Então, o que é o ELK Stack? "ELK" é a sigla para três projetos de código aberto: Elasticsearch, Logstash e Kibana. O Elasticsearch é um mecanismo de pesquisa e análise. O Logstash é um pipeline de processamento de dados do lado do servidor que ingere dados de várias fontes simultaneamente, os transforma e os envia para um "stash" como o Elasticsearch.
O Kibana permite que os usuários visualizem dados com tabelas e gráficos no Elasticsearch.

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

Agora acesse o kibana <your-elasticsearch-server>:5601/app/kibana#/management/kibana/objects?_g=(refreshInterval:(pause:!f,value:5000),time:(from:now-15m,to:now)) e importe os arquivos de objetos salvos (nesta ordem)

- searches-base.json
- visualisations-base.json
- dashboards-base.json
