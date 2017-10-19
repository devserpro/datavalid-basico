# Datavalid - Pacote Básico <span id="trialSpan"></span>

Solução de validação de informações que garante autenticidade e confiabilidade aos dados em tempo real.

O DataValid é disponibilizado pela plataforma APIGOV (Plataforma que contempla todas as API's disponibilizadas e comercializadas pelo SERPRO) e utiliza o protocolo Oauth2 - Client Credential Grant ([https://tools.ietf.org/html/rfc6749#section-4.4](https://tools.ietf.org/html/rfc6749#section-4.4)) para realizar a autenticação e autorização de acesso para consumo das API's contratadas, conforme figura abaixo:

<img title="Processo de autenticação e autorização APIS" src="https://raw.githubusercontent.com/devserpro/consulta-cpf/master/img/oauth.png" style="width=50%;" />

## Como fazer consultas ao Datavalid

Para consumir o Datavalid, você deverá utilizar os dois códigos (Consumer Key e Consumer Secret) disponibilizados na Área do Cliente. Esses códigos servem para identificar o contrato e deverão ser informados sempre que uma consulta for realizada.
Exemplos de códigos: 

**Consumer Key**: djaR21PGoYp1iyK2n2ACOH9REdUb

**Consumer Secret**: ObRsAJWOL4fv2Tp27D1vd8fB3Ote

### 1 – Como solicitar o Token de Acesso (Bearer)
Para consultar a Datavalid, é necessário obter um token de acesso temporário (Bearer). Esse token possui um tempo de validade e sempre que expirado, este passo de requisição de um novo token de acesso deve ser repetido. 

Para solicitar o token temporário é necessário realizar uma requisição HTTP POST para o endpoint Token https://apigateway.serpro.gov.br/token, informando as credenciais de acesso(consumerKey:consumerSecret) no HTTP Header Authorization, no formato base64, conforme exemplo abaixo. As credenciais de acesso devem ser obtidas a partir do portal do cliente Serpro - https://minhaconta.serpro.gov.br

```
[POST] grant_type=client_credentials
[HEAD] Authorization: Basic base64(Consumer Key:Consumer Secret)
```

Abaixo segue um exemplo de chamada via cUrl:

```curl
curl -k -d "grant_type=client_credentials" -H "Authorization: Basic ZGphUjIxUEdvWXAxaXlLMm4yQUNPSDlSRWRVYjpPYlJzQUpXT0w0ZnYyVHAyN0QxdmQ4ZkIzT3RlCg" https://apigateway.serpro.gov.br/token
```

A chave informada no exemplo acima "ZGphUjIxUEdvWXAxaXlLMm4yQUNPSD
lSRWRVYjpPYlJzQUpXT0w0ZnYyVHAyN0QxdmQ4ZkIzT3RlCg" é resultado do BASE64 dos códigos Consumer Key e Consumer Secret separados pelo caracter “:”, conforme exemplo a seguir:

```curl
echo -n "djaR21PGoYp1iyK2n2ACOH9REdUb:ObRsAJWOL4fv2Tp27D1vd8fB3Ote" | base64
```

**Receba o Token**

Como resultado, o endpoint informará o token de acesso a API, no campo access_token da mensagem json de retorno. Este token deve ser informado nos próximos passos.

```json
{"scope":"am_application_scope default","token_type":"Bearer","expires_in":3295,"access_token":"c66a7def1c96f7008a0c397dc588b6d7"}
```

**Renovação do Token de Acesso**

Atentar que sempre que o token de acesso temporário expirar, o gateway vai retornar um _HTTP CODE 401_ após realizar uma requisição para uma API. Neste caso, deve ser repetido o passo anterior (**Como solicitar o Token de Acesso (Bearer)**) para geração de um novo token de acesso temporário.


### 2 – Como realizar a consulta ao Datavalid

De posse do Token de Acesso, faça a requisição a um dos serviços do DataValid. Exemplo:

```curlBearer
curl -X POST "https://apigateway.serpro.gov.br/datavalid-trial/basico/vbeta1/validate/pf" -H  "accept: application/json" -H  "Authorization: Bearer c66a7de41c96f7008a0c397dc588b6d7" -H  "content-type: application/json" -d "{  \"key\": {    \"cpf\": \"05137518743\"  },  \"answer\": {    \"nome\": \"João\",    \"sexo\": \"F\",    \"data_nascimento\": \"2000-10-10\",    \"situacao_cpf\": \"regular\",    \"filiacao\": {      \"nome_mae\": \"Mãe do João\",      \"nome_pai\": \"Pai do João\"    },    \"nacionalidade\": 1,    \"endereco\": {      \"logradouro\": \"Nome do Lograudoro\",      \"numero\": \"0007\",      \"complemento\": \"APTO 2015\",      \"bairro\": \"Nome do Bairro\",      \"cep\": \"0000001\",      \"municipio\": \"Nome do Municipio\",      \"uf\": \"DF\"    },    \"documento\": {      \"tipo\": 1,      \"numero\": \"000001\",      \"orgao_expedidor\": \"SSP\",      \"uf_expedidor\": \"MG\"    },    \"cnh\": {      \"numero_registro\": \"0000001\",      \"categoria\": \"AB\",      \"data_primeira_habilitacao\": \"2000-10-10\",      \"data_validade\": \"2000-10-10\"    }  }}"
```

No exemplo acima foram utilizados os seguintes parametros:

**[HEADER] Accept: application/json** - Informamos o tipo de dados que estamos requerendo, nesse caso JSON

**[HEADER] Authorization: Bearer <span class="bearer">c66a7de41c96f7008a0c397dc588b6d7</span>** - Informamos o token de acesso recebido

**[POST] https://apigateway.serpro.gov.br/datavalid-trial/basico/vbeta1/validate/pf**: chamamos a url do serviço de validação de PF do Datavalid passando como argumento -d, o corpo da requisiço REST."

Exemplo de resposta para validação de dados de PF (Pessoa Física):

```json
{
    "nome": false,
    "nome_similaridade": 0.1923076923076923,
    "sexo": true,
    "data_nascimento": false,
    "situacao_cpf": true,
    "filiacao": {
        "nome_mae": false,
        "nome_mae_similaridade": 0.27586206896551724,
        "nome_pai": false,
        "nome_pai_similaridade": 0.21739130434782605
    },
    "nacionalidade": true,
    "endereco": {
        "logradouro": false,
        "logradouro_similaridade": 0.15384615384615385,
        "numero": false,
        "numero_similaridade": 0.16666666666666663,
        "complemento": false,
        "complemento_similaridade": 0.09999999999999998,
        "bairro": false,
        "bairro_similaridade": 0.16666666666666663,
        "cep": false,
        "municipio": false,
        "municipio_similaridade": 0.15384615384615385,
        "uf": false,
        "uf_similaridade": 0
    },
    "documento": {
        "numero": false,
        "orgao_expedidor": false,
        "uf_expedidor": true
    }
}
```

Exemplo de resposta para validação de dados de PJ (Pessoa Jurídica):

```json
{
    "razao_social": false,
    "razao_social_similaridade": 0.96,
    "nome_fantasia": true,
    "nome_fantasia_similaridade": 1,
    "data_abertura": true,
    "cnae_principal": {
        "codigo": true,
        "descricao": true,
        "descricao_similaridade": 1
    },
    "natureza_juridica": {
        "codigo": true,
        "descricao": true,
        "descricao_similaridade": 1
    },
    "endereco": {
        "logradouro": false,
        "logradouro_similaridade": 0.9130434782608696,
        "numero": true,
        "numero_similaridade": 1,
        "complemento": false,
        "complemento_similaridade": 0,
        "bairro": true,
        "bairro_similaridade": 1,
        "cep": true,
        "municipio": true,
        "municipio_similaridade": 1,
        "uf": true,
        "uf_similaridade": 1
    },
    "situacao_cadastral": {
        "codigo": true,
        "data": true,
        "motivo": false,
        "motivo_similaridade": 0
    },
    "situacao_especial": true,
    "situacao_especial_similaridade": 1,
    "nome_orgao": true,
    "nome_orgao_similaridade": 1,
    "ente_federativo": true,
    "ente_federativo_similaridade": 1,
    "correio_eletronico": true,
    "correio_eletronico_similaridade": 1,
    "capital_social": true,
    "porte": true,
    "telefone": {
        "ddd": true,
        "numero": true
    }
}
```
Abaixo disponibilizamos body's de requisições que podem ser utilizados na demonstração dos metodos do Datavalid. Esses body's possuem dados fictícios e podem ser usados para simular cenários nos quais o Datavalid valida positivamente todos os dados enviados.

**Validação de PF (validate/pf) :**
```json

{  
   "key":{  
      "cpf":"81845995104"
   },
   "answer":{  
      "nome":"ADEMAR VEGA XIMENES",
      "sexo":"M",
      "data_nascimento":"1994-06-23",
      "situacao_cpf":"REGULAR",
      "filiacao":{  
         "nome_mae":"MARIA VEGA XIMENES",
         "nome_pai":"JOÃO VEGA XIMENES"
      },
      "endereco":{  
         "logradouro":"Travessa Serrano",
         "numero":"9754",
         "complemento":"",
         "cep":"12983-406",
         "bairro":"CENTRO",
         "municipio":"Nova Iguaçu",
         "uf":"AC"
      },
      "nacionalidade":1,
      "documento":{  
         "tipo":1,
         "numero":"6694845",
         "orgao_expedidor":"DIC",
         "uf_expedidor":"MA"
      },
      "cnh":{  
         "numero_registro":"98668270420",
         "categoria":"A",
         "data_primeira_habilitacao":"1980-11-28",
         "data_validade":"2018-09-02"
      }
   }
}

```

**Validação de PJ (validate/pj) :**
```json
{
   "key":{
      "cnpj":"34238864000168"
   },
   "answer":{
      "razao_social":"SERVICO DE E-COMERCE LTDA",
      "nome_fantasia":"E-COMERCE",
      "data_abertura":"1967-06-30",
      "cnae_principal":{
         "codigo":"6204000",
         "descricao":"Consultoria em e-comerce"
      },
      "natureza_juridica":{
         "codigo":"2011",
         "descricao":"Empresa Privada"
      },
      "endereco":{
         "logradouro":"ST DE GRANDE AREA NORTE",
         "numero":"Q.601",
         "complemento":"LOTE V",
         "cep":"70836900",
         "bairro":"ASA NORTE",
         "municipio":"BRASILIA",
         "uf":"DF"
      },
      "situacao_especial":"",
      "situacao_cadastral":{
         "codigo":2,
         "data":"2004-05-22",
         "motivo":""
      },
      "nome_orgao":"BRASILIA",
      "ente_federativo":"BR",
      "correio_eletronico":"",
      "capital_social":0,
      "porte":"05",
      "telefone":{
         "ddd":"61",
         "numero":"4338456"
      }
   }
}
```

**Conjunto de dados falsos de Pessoas Fisicas** 

Para facilitar a simulaço de diversos cenários de validação de Pessoas Fisicas, disponibilizamos abaixo
dados falsos de Pessoas Fisicas que podem ser utilizados nesse ambiente de demonstração. 

([Link para Github](https://github.com/devserpro/datavalid-basico/blob/master/_layouts/DadosPessoasFisicas.csv))

([Download direto do arquivo](https://rawgit.com/devserpro/datavalid-basico/master/_layouts/DadosPessoasFisicas.csv)) 

