# Implantação da solução

# Implantação da Solução em Ambiente de Computação em Nuvem

A aplicação de previsão de demanda foi implantada em um ambiente de computação em nuvem utilizando os serviços da plataforma **Microsoft Azure**, por meio do endereço público:
**https://pucwebaplicacaog4.azurewebsites.net/**

A escolha da Azure se deu pela integração nativa com aplicações Python, facilidade de publicação contínua e ferramentas de monitoramento integradas, o que permite supervisionar o desempenho da aplicação e escalar os recursos sempre que necessário.

---

## Planejamento da Capacidade Operacional

Antes da implantação, foi realizada uma estimativa de carga com base em modelagem matemática e simulação, considerando o comportamento de uso esperado da aplicação.

### 1. Volume de Requisições

O modelo CatBoost executa previsões rapidamente, com tempo médio de resposta inferior a poucos milissegundos por requisição.  
Mesmo com um número moderado de usuários simultâneos (entre 5 e 50), a aplicação opera confortavelmente em uma instância básica do Azure App Service.

### 2. Modelagem Matemática da Capacidade

Para estimar o comportamento sob carga, foi utilizado o conceito de sistemas de filas baseado na fórmula de Erlang-C para estimativa de tempo de resposta em sistemas M/M/1 — suficientes para projetos desse porte.
O serviço se comporta como um sistema de fila com taxa média de chegada λ e taxa média de atendimento μ, sendo μ muito superior a λ devido à baixa complexidade computacional do modelo.
Assim, a taxa de ocupação ρ = λ/μ se mantém muito abaixo de 0,7, o que indica que a latência se mantém estável mesmo em cenários de pico.

### 3. Simulação do Sistema

Para validar o modelo matemático, foi utilizada uma simulação de carga com ferramentas de teste como Locust e Apache Bench, enviando diversas requisições simultâneas à API.
O comportamento observado foi coerente com o modelo teórico, apresentando estabilidade e mantendo a média de tempo de resposta adequada.
A partir disso, definiu-se que uma única instância com autoscaling habilitado seria suficiente para absorver variações de demanda.
---

## Escolha e Configuração do Provedor de Nuvem

A solução foi hospedada no **Azure App Service**, que oferece:

- Suporte nativo a Python e FastAPI  
- Publicação simplificada  
- Gerenciamento automático de infraestrutura  
- Escalabilidade horizontal configurável  
- Monitoramento via Application Insights  

A aplicação foi configurada em ambiente **Linux**, utilizando runtime **Python 3.x**, com plano básico capaz de atender à demanda prevista.

---

## Empacotamento da Aplicação

A aplicação foi empacotada seguindo boas práticas para APIs Python:

1. **Estrutura de diretórios**  
   - `main.py`: rotas FastAPI e carregamento do modelo  
   - `modelo_catboost_grupo4.cbm`: modelo CatBoost treinado  
   - `requirements.txt`: dependências do ambiente

2. **Comando de inicialização**  
   O App Service executa a API via Gunicorn:
gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app

3. **Configurações adicionais**  
- Habilitação de CORS para permitir acesso ao front-end  
- Variáveis de ambiente configuradas no painel da Azure  

---

## Publicação da Aplicação na Azure

O processo de publicação seguiu as etapas:

1. Criação do recurso **App Service** no portal Azure  
2. Seleção de ambiente Linux e runtime Python  
3. Deploy da aplicação via **ZIP Deploy**, **FTP** ou **GitHub Actions**  
4. Configuração do arquivo de requisitos e startup  
5. Validação do funcionamento no endpoint público  
6. Teste completo com o front-end usando requisições via fetch  

Após a publicação, o serviço passou a responder no endpoint:
https://pucwebaplicacaog4.azurewebsites.net/predict
Permitindo que qualquer cliente HTTP ou interface web faça previsões sem depender da execução local da API.



# Apresentação da solução

Link Serviço Servidor Azure: https://pucwebaplicacaog4.azurewebsites.net/ 
Link API: https://pucwebaplicacaog4.azurewebsites.net/predict 
Link Apresentação: https://pucwebaplicacaog4.azurewebsites.net/ApresentaçãoVersãoFinal.html 


Como reproduzir
uv pip install -r src/api/requirements.txt
uv run uvicorn main:app --host 0.0.0.0 --port 8000 --reload
Abrir swagger no browser em http://127.0.0.1:8000/docs
Exemplo de payload para o endpoint /predict:
{
  "ship_mode":    "Standard Class",
  "customer_id":  "DB-12970",
  "segment":      "Corporate",
  "country":      "United States",
  "city":         "Houston",
  "state":        "Texas",
  "postal_code":  77036,
  "region":       "Central",
  "category":     "Office Supplies",
  "sub_category": "Art",
  "sales":        2.672,
  "ano":          2015,
  "mes":          11,
  "dia_semana":   "Quinta-feira",
  "day_of_week":  "Thursday"
}



