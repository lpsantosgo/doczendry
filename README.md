Uma breve introdução

Seja bem-vindo(a)!

Aqui você encontrará tudo que precisa para ter sua melhor integração possível com as nossas APIs. Venha conosco e conduza seu negócio ao próximo nível! 🚀
O que é a Zendry ?

Desde o nosso início em 2018, como Facilitadora de Pagamentos pela Instituição financeira ADIQ Pagamentos, nos dedicamos a fornecer serviços seguros, transparentes e ágeis para facilitar transações financeiras no ambiente online.

Em 2019 expandimos nosso escopo ao lançar a conta digital e iniciar o processamento, gerenciamento e automação de pagamentos para plataformas de e-commerce.

Os serviços oferecidos pela Zendry se caracterizam por fornecerem aos parceiros uma forma de comunicação com instituições de pagamentos, atuando como uma “ponte” neste sentido, preservando sempre a segurança e confidencialidade dos dados por ela trafegados. Tem como objetivo permitir a consulta e execução de operações financeiras, tornando-se assim uma ferramenta de gestão para o parceiro Zendry. Qualquer transação efetuada através da Zendry está sujeita e deve estar em conformidade com as leis da República Federativa do Brasil. Aconselhamos que você leia os termos e condições cuidadosamente.
Visão geral da documentação
A API de integração está implementada em conformidade com o princípio de design REST. Nossa API possui recursos orientados a URLs, com códigos HTTP para indicar erros. Nós utilizamos funcionalidades HTTP nativas, como o verbo de ação GET para operações de leitura, bem como modelo de autenticação baseado em token. Toda a comunicação deve ser criptografada via SSL/HTTPS. A aplicação cliente deve enviar um cabeçalho de Autorização HTTP contendo o token gerado a cada solicitação.

Authentication
Obtenha suas credenciais

Para obter suas credenciais, basta solicitá-las ao nosso time de suporte informando o seu CNPJ, Razão Social e qual produto deseja testar. Caso o produto seja Pix ou Baas, para receber o retorno dos status, favor informar também a URL, Login e Senha do Webhook referente ao método de autenticação Basic.
Ambiente de Produção

Para acessar os endpoints em ambiente de produção, utilize a seguinte URL base:
BASEURL

https://api.zendry.com.br

Acessando a API

Nossa API utiliza sistema de autenticação OAuth2.0. Portanto, o processo de autorização será composto por duas etapas. A geração do token de acesso e o seu envio nos Headers das requisições gerais da API.

Para gerar o token, deverá ser utilizado as credenciais CLIENT_ID e CLIENT_SECRET fornecidos pela equipe de tecnologia da

Geração do Token de Acesso
Como Gerar o Token de Acesso

Para gerar o Token de Acesso, é necessário realizar uma requisição utilizando HTTP Basic Authentication através de um método POST, de modo que seja enviado um Header Authorization com valor Basic {CODE_BASE64}. As credenciais CLIENT_ID e CLIENT_SECRET devem ser concatenadas com um símbolo “:” para que então sejam informadas como valor para o campo citado.

Deve também ser informado o tipo de autenticação no corpo da mensagem como client_credentials. Segue o modelo de requisição utilizado para gerar o token de acesso:
URL AUTH

  BASEURL/auth/generate_token

HTTP Headers - Exemplo

  Authorization: BASIC base_64({CLIENT_ID}:{CLIENT_SECRET})
  Content-Type: application/json

HTTP Request Body

  {
    "grant_type": "client_credentials"
  }

Caso os dados informados estejam corretos, será retornada uma mensagem HTTP 200 - OK informando o código gerado e o seu tempo de validade. Esse código deverá ser utilizado em todas as requisições posteriores, e precisará ser gerado novamente após atingir seu tempo de expiração.
HTTP 200 Response Body - Exemplo

  {    
    "access_token": "eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoxLCJleHAiOjE2NjUwNjc2MDJ9.ogQs-M3JPH9kgQuqOVc9erv3kBlXr5i2Y7bDFjNh3qI",
    "token_type": "Bearer",
    "expires_in": 1800
  }

Descrição dos atributos
Atributo	Descrição	Tipo
access_token
(Obrigatório)	Token JWT a ser utilizado nas demais requisições da API	STRING
token_type
(Obrigatório)	Tipo de token gerado	STRING
expires_in
(Obrigatório)	Tempo de validade do token, em segundos	INTEGER

Em caso de invalidade nas informações passadas na requisição para criação de token, será retornado HTTP 401 - Unauthorized, de acordo com o exemplo abaixo.
HTTP 401 Response Body - Exemplo

  {
    "error": "Unauthorized"
  }

Autorização de Endpoint

As demais requisições da API deverão ser realizadas utilizando o token gerado, precedido do tipo ao qual é correspondente. Ex: Bearer {TOKEN}
HTTP Headers - Exemplo

  Authorization: {TOKEN_TYPE} {ACCESS_TOKEN}
  Content-Type: application/json
  
  Gerar QRCode de Cobrança

A API de recebimentos Pix foi implementada para que o parceiro pudesse fazer uma gestão de cobranças pela ferramenta Pix, gerando QrCodes e consultando-os posteriormente para visualizar os detalhes do pagamento. Uma vez que uma cobrança é sinalizada como paga, a API envia uma mensagem para notificar o parceiro através de um Webhook previamente cadastrado. Mais detalhes poderão ser encontrados na seção Webhooks deste documento.
Gerar QRCode de Cobrança

Este endpoint realiza a geração de um QRCode de pagamento Pix. Para fazê-lo deve ser efetuada a chamada para a API, como especificado abaixo:
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

  /v1/pix/qrcodes

HTTP Request Body

{
    "value_cents": 10001 ,
    "generator_name": "John Doe" ,
    "generator_document": "12345678900",
    "expiration_time": "1800",
    "external_reference": "Teste2"
}

Descrição dos atributos
ATRIBUTOS	DESCRIÇÃO	TIPO
value_cents
(Obrigatório)	Valor, em centavos, da cobrança	INTEGER
maior que 0
generator_name
(Opcional)	Nome do usuário gerador do qrcode. Será utilizado para cadastrar a cobrança.	STRING
limite de 100 caracteres
generator_document
(Opcional)	Documento (CPF/CNPJ) do usuário gerador do qrcode. Será utilizado para cadastrar a cobrança. Obrigatório caso seja informado o atributo generator_name.	STRING
limite de 14 caracteres
Apenas números
expiration_time
(Opcional)	Tempo de expiração do QRCode gerado para cobrança, em segundos.	INTEGER
maior que 0
external_reference
(Opcional)	Valor informado pelo parceiro como identificação do qrcode gerado.	STRING
limite de 255 caracteres
Sucesso

Após a chamada, é retornado um JSON com o status 201 - Created caso o procedimento tenha ocorrido com sucesso.

cont ssas = Zendry
HTTP 201 Response Body - Exemplo

{
    "qrcode": {
        "reference_code": "ZENDRYTESTPIXQRCODE6" {{ssas}},
        "external_reference": "Teste2",
        "content": "00000000000000000000br.gov.bcb.pix012345678900-1234-1234-1234-12345678901234567890123456789000.012345BR5925Zendry Solucoes em Paga6009SAO PAULO62230519ZENDRYPIXQRCODETESTE00000",
        "image_base64": null
    }
}

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
qrcode
(Obrigatório)	Objeto de dados do qrcode cadastrado.	OBJECT
qrcode.external_reference
(Opcional)	Valor informado pelo parceiro como identificação do qrcode gerado.	STRING
limite de 255 caracteres
qrcode.reference_code
(Obrigatório)	Identificador único do QRCode.	STRING
limite de 100 caracteres
qrcode.content
(Obrigatório)	Conteúdo do QRCode. (Código copia e cola Pix)	STRING
limite de 255 caracteres
padrão: 30
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

  {
    "error": "generator_document not_a_number | generator_document required | generator_name required"
  }

HTTP 403 Response Body - Exemplo

  {
    "error": "Generator not allowed"
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
  
  Gerar Checkout de Cobrança

Este endpoint realiza a geração de um link para uma página de cobrança. Para fazê-lo deve ser efetuada a chamada para a API, como especificado abaixo:
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

  /v1/pix/checkout

HTTP Request Body

{
    "value_cents": 10001 ,
    "generator_name": "John Doe" ,
    "generator_document": "12345678900",
    "expiration_time": "1800",
    "external_link": "johndoepage.com"
}

Descrição dos atributos
ATRIBUTOS	DESCRIÇÃO	TIPO
value_cents
(Obrigatório)	Valor, em centavos, da cobrança	INTEGER
maior que 0
generator_name
(Obrigatório)	Nome do usuário gerador do qrcode. Será utilizado para cadastrar a cobrança.	STRING
limite de 100 caracteres
generator_document
(Obrigatório)	Documento (CPF/CNPJ) do usuário gerador do qrcode. Será utilizado para cadastrar a cobrança. Obrigatório caso seja informado o atributo generator_name.	STRING
limite de 14 caracteres
Apenas números
expiration_time
(Opcional)	Tempo de expiração do QRCode gerado para cobrança, em segundos	INTEGER
maior que 0
Se não informado, padrão: 1800
external_link
(Obrigatório)	Endereço web do parceiro, para a qual o usuário poderá retornar.	STRING
limite de 255 caracteres
Sucesso

Após a chamada, é retornado um JSON com o status 201 - Created caso o procedimento tenha ocorrido com sucesso.
HTTP 201 Response Body - Exemplo

{
    "qrcode_checkout": {
        "payment_link": "https://checkout.zendry.com.br/pix_charge/6058042c-53d9-412c-9bbe-41032a8acc99",
        "request_id": "6058042c-53d9-412c-9bbe-41032a8acc99",
        "reference_code": "ZENDRYTESTPIXQRCODE734"
    }
}

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
qrcode_checkout
(Obrigatório)	Objeto de dados do checkou de qrcode cadastrado.	OBJECT
qrcode_checkout.payment_link
(Obrigatório)	Link da página de pagamento do QR Code.	STRING
limite de 255 caracteres
qrcode_checkout.request_id
(Obrigatório)	Identifica o código do link de pagamento. É o identificador do checkout no link gerado	STRING
limite de 255 caracteres
qrcode_checkout.reference_code
(Obrigatório)	Identificador único do QRCode.	STRING
limite de 100 caracteres
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

  {
    "error": "generator_document not_a_number | generator_document required | generator_name required"
  }

HTTP 403 Response Body - Exemplo

  {
    "error": "Generator not allowed"
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
  
  Listar QRCodes

Permite obter uma lista de cobranças geradas atribuindo diversos filtros para selecionar os itens desejados. A resposta vem em formato paginado, sendo no máximo 100 registros por página. Um atributo meta será retornado informando os parâmetros totalizadores e o total de páginas da consulta realizada.
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  /v1/pix/qrcodes

URL Params – Filter Options
PARÂMETRO	DESCRIÇÃO	TIPO
page
(Opcional)	Número da página. Se não informado, assume o valor 1	INTEGER
maior que 0
registration_start_date
(Obrigatório caso registration_end_date seja informado)	Valor inicial para o período de busca por data de geração das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
registration_end_date
(Obrigatório caso registration_start_date seja informado)	Valor final para o período de busca por data geração das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
payment_start_date
(Obrigatório caso payment_end_date seja informado)	Valor inicial para o período de busca por data de pagamento das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
payment_end_date
(Obrigatório caso payment_start_date seja informado)	Valor final para o período de busca por data pagamento das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
generator_name
(Opcional)	Nome do gerador das cobranças. Consulta do tipo LIKE	STRING
limite 100 caracteres
generator_document
(Opcional)	CPF/CNPJ do gerador das cobranças. Consulta do tipo LIKE	STRING
limite 14 caracteres apenas números
payer_name
(Opcional)	Nome do pagador das cobranças. Consulta do tipo LIKE	STRING
limite 100 caracteres
payer_document
(Opcional)	CPF/CNPJ do pagador das cobranças. Consulta do tipo LIKE	STRING
limite 14 caracteres apenas números
status
(Opcional)	Status das cobranças	STRING (Enum)
error (Erro ao gerar cobrança)
awaiting_payment (Aguardando Pagamento)
paid (Pago)
canceled (Cancelado)
reference_code
(Opcional)	Código de identificação. Consulta do tipo LIKE	STRING
limite 100 caracteres
end_to_end
(Opcional)	Código de pagamento. Consulta do tipo LIKE	STRING
limite 32 caracteres
Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

{
    "qrcodes": [
        {
            "reference_code": "ZENDRYTESTPIXQRCODE1",
            "external_reference": null,
            "value_cents": 1,
            "content": "00000000000000000000br.gov.bcb.pix012345678900-1234-1234-1234-12345678901234567890123456789000.012345BR5925Zendry Solucoes em Paga6009SAO PAULO62230519ZendryPIXQRCODETESTE00000",
            "status": "awaiting_payment",
            "generator_name": null,
            "generator_document": null,
            "payer_name": null,
            "payer_document": null,
            "registration_date": "2024-04-02T14:24:36.000-03:00",
            "payment_date": "2024-04-02T14:24:36.000-03:00",
            "end_to_end": null
        }
    ],
    "meta": {
        "current_page": 1,
        "total_pages": 1,
        "total_items_amount": 6,
        "total_value_cents": 50004
    }
}

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
qrcodes
(Obrigatório)	Lista de cobranças qrcodes que obedecem os filtros especificados	ARRAY
qrcodes.reference_code
(Obrigatório)	Identificador único do QRCode.	STRING
limite de 100 caracteres
qrcode.external_reference
(Opcional)	Valor informado pelo parceiro como identificação do qrcode gerado.	STRING
limite de 255 caracteres
qrcodes.value_cents
(Obrigatório)	Valor da cobrança, em centavos	INTEGER
Maior que 0
qrcodes.content
(Obrigatório)	Conteúdo do QRCode. (Código copia e cola Pix)	STRING
limite de 255 caracteres
qrcodes.status
(Obrigatório)	Status da cobrança	ENUM
error (Erro ao gerar cobrança)
awaiting_payment(Aguardando Pagamento)
paid (Pago)
canceled (Cancelado)
qrcodes.generator_name
(Opcional)	Nome do pagador	STRING
limite de 100 caracteres
qrcodes.generator_document
(Opcional)	Documento do gerador	STRING
limite de 14 caracteres
qrcodes.payer_name
(Opcional)	Nome do pagador	STRING
limite de 100 caracteres
qrcodes.payer_document
(Opcional)	Documento do pagador	STRING
limite de 14 caracteres
qrcodes.registration_date
(Obrigatório)	Data de criação da cobrança	STRING
formato datetime YYYY-mm-ddTHH:MM:ss.z
qrcodes.payment_date
(Opcional)	Data de pagamento da cobrança	STRING
formato datetime YYYY-mm-ddTHH:MM:s s.z
qrcodes.end_to_end
(Opcional)	Identificador único do pagamento Pix	STRING
limite de 32 caracteres
meta
(Obrigatório)	Objeto que contém os metadados da requisição	OBJECT
meta.current_page
(Obrigatório)	Página atual da consulta	INTEGER
maior que 0
meta.total_pages
(Obrigatório)	Total de páginas da consulta realizada	INTEGER
maior que 0
meta.total_items_amount
(Obrigatório)	Quantidade total de cobranças existentes considerando todas as páginas	INTEGER
maior que 0
meta.total_value_cents
(Obrigatório)	Valor total, em centavos das cobranças considerando todas as páginas	INTEGER
maior que 0
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

{
  "error": "registration_end_date required"
}

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }


Consulta QRCode - Código Referência

Similar ao endpoint de consulta de qrcode por código de referência, desta vez sendo necessário informar como parâmetro o código do pagamento Pix efetuado pelo cliente (end_to_end_pgto)
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  /v1/pix/qrcodes/{reference_code}

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

{
    "qrcode": {
        "reference_code": "ZENDRYTESTPIXQRCODE1",
        "external_reference": null,
        "value_cents": 1,
        "content": "00000000000000000000br.gov.bcb.pix012345678900-1234-1234-1234-12345678901234567890123456789000.012345BR5925Zendry Solucoes em Paga6009SAO PAULO62230519ZENDRYPIXQRCODETESTE00000",
        "status": "awaiting_payment",
        "generator_name": null,
        "generator_document": null,
        "payer_name": null,
        "payer_document": null,
        "payer_bank_name": null,
        "payer_agency": null,
        "payer_account": null,
        "payer_account_type": null,
        "registration_date": "2024-04-02T14:24:36.000-03:00",
        "payment_date": "2024-04-02T14:24:36.000-03:00",
        "end_to_end": null
    }
}

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
qrcode
(Obrigatório)	Objeto de dados do qrcode cadastrado.	OBJECT
qrcode.reference_code
(Obrigatório)	Identificador único do QRCode.	STRING
limite de 100 caracteres
qrcode.external_reference
(Opcional)	Valor informado pelo parceiro como identificação do qrcode gerado.	STRING
limite de 255 caracteres
qrcode.content
(Obrigatório)	Conteúdo do QRCode. (Código copia e cola Pix)	STRING
limite de 255 caracteres
qrcode.value_cents
(Obrigatório)	Valor da cobrança, em centavos	INTEGER
Maior que zero
qrcode.status
(Obrigatório)	Status da cobrança	ENUM
error (Erro ao gerar cobrança)
awaiting_payment(Aguardando Pagamento)
paid (Pago)
canceled (Cancelado)
qrcode.generator_name
(Opcional)	Nome do gerador	STRING
limite de 100 caracteres
qrcode.generator_document
(Opcional)	Documento do gerador	STRING
limite de 14 caracteres
qrcode.payer_name
(Opcional)	Nome do pagador	STRING
limite de 100 caracteres
qrcode.payer_document
(Opcional)	Documento do pagador	STRING
limite de 14 caracteres
qrcode.payer_bank_name
(Opcional)	Nome da instituição financeira do pagador	STRING
limite de 50 caracteres
qrcode.payer_agency
(Opcional)	Agência da conta bancária do pagador	STRING
limite de 20 caracteres
qrcode.payer_account
(Opcional)	Número da conta bancária do pagador	STRING
limite de 20 caracteres
qrcode.payer_account_type
(Opcional)	Tipo de conta bancária do pagador	ENUM
checking_account
savings_account
salary_account
transaction_account
qrcodes.registration_date
(Obrigatório)	Data de criação da cobrança	STRING
formato datetime YYYY-mm-ddTHH:MM:ss.z
qrcode.payment_date
(Opcional)	Data de pagamento da cobrança	STRING
formato datetime YYYY-mm-ddTHH:MM:ss.z
qrcode.end_to_end
(Opcional)	Identificador único do pagamento Pix.	STRING
limite de 32 caracteres
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 406 Response Body - Exemplo

  {
    "error": "Qrcode not found."
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
  
  Consulta QRCode - EndtoEnd

Similar ao endpoint de consulta de qrcode por código de referência, desta vez sendo necessário informar como parâmetro o código do pagamento Pix efetuado pelo cliente (end_to_end_pgto).
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  /v1/pix/qrcodes/end_to_end/{end_to_end_pgto}/

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

{
    "qrcode": {
        "reference_code": "ZENDRYTESTPIXQRCODE1",
        "external_reference": null,
        "value_cents": 1,
        "content": "00000000000000000000br.gov.bcb.pix012345678900-1234-1234-1234-12345678901234567890123456789000.012345BR5925Zendry Solucoes em Paga6009SAO PAULO62230519ZENDRYPIXQRCODETESTE00000",
        "status": "awaiting_payment",
        "generator_name": null,
        "generator_document": null,
        "payer_name": null,
        "payer_document": null,
        "payer_bank_name": null,
        "payer_agency": null,
        "payer_account": null,
        "payer_account_type": null,
        "registration_date": "2024-04-02T14:24:36.000-03:00",
        "payment_date": "2024-04-02T14:24:36.000-03:00",
        "end_to_end": null
    }
}

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
qrcode
(Obrigatório)	Objeto de dados do qrcode cadastrado.	OBJECT
qrcode.reference_code
(Obrigatório)	Identificador único do QRCode.	STRING
limite de 100 caracteres
qrcode.external_reference
(Opcional)	Valor informado pelo parceiro como identificação do qrcode gerado.	STRING
limite de 255 caracteres
qrcode.content
(Obrigatório)	Conteúdo do QRCode. (Código copia e cola Pix)	STRING
limite de 255 caracteres
qrcode.value_cents
(Obrigatório)	Valor da cobrança, em centavos	INTEGER
Maior que zero
qrcode.status
(Obrigatório)	Status da cobrança	ENUM
error (Erro ao gerar cobrança)
awaiting_payment(Aguardando Pagamento)
paid (Pago)
canceled (Cancelado)
qrcode.generator_name
(Opcional)	Nome do gerador	STRING
limite de 100 caracteres
qrcode.generator_document
(Opcional)	Documento do gerador	STRING
limite de 14 caracteres
qrcode.payer_name
(Opcional)	Nome do pagador	STRING
limite de 100 caracteres
qrcode.payer_document
(Opcional)	Documento do pagador	STRING
limite de 14 caracteres
qrcode.payer_bank_name
(Opcional)	Nome da instituição financeira do pagador	STRING
limite de 50 caracteres
qrcode.payer_agency
(Opcional)	Agência da conta bancária do pagador	STRING
limite de 20 caracteres
qrcode.payer_account
(Opcional)	Número da conta bancária do pagador	STRING
limite de 20 caracteres
qrcode.payer_account_type
(Opcional)	Tipo de conta bancária do pagador	ENUM
checking_account
savings_account
salary_account
transaction_account
qrcodes.registration_date
(Obrigatório)	Data de criação da cobrança	STRING
formato datetime YYYY-mm-ddTHH:MM:ss.z
qrcode.payment_date
(Opcional)	Data de pagamento da cobrança	STRING
formato datetime YYYY-mm-ddTHH:MM:ss.z
qrcode.end_to_end
(Opcional)	Identificador único do pagamento Pix.	STRING
limite de 32 caracteres
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 406 Response Body - Exemplo:

  {
    "error": "Qrcode not found."
  }

HTTP 500 Response Body - Exemplo:

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
Pagar QRCode

Realiza a simulação de pagamento de um QRCode, em homologação.

Quando o QRCode for pago, será efetuado o envio da mensagem de notificação via webhook cadastrado.
Atenção

Não está disponível em produção
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

  /v1/pix/qrcodes/{reference_id}/pay

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

  {
    "message": "Operation succeeded"
  }

Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 406 Response Body - Exemplo

  {
    "error": "Qrcode not found"
  }

HTTP 422 Response Body - Exemplo

  {
    "error": "Qrcode canceled"
  }
  Cadastrar Pagamentos

A API de pagamentos Pix foi implementada para permitir que o parceiro possa solicitar transferências bancárias à partir da ferramenta Pix, tornando possível o acompanhamento posterior dessas solicitações. Uma vez que uma alteração de status é identificada em cada um dos pagamentos registrados pelo parceiro, a API envia uma mensagem para notificar o parceiro através de um Webhook previamente cadastrado. Mais detalhes poderão ser encontrados na seção Webhooks deste documento.
Cadastrar Pagamentos

Realiza o cadastro de um pagamento Pix. Deverão ser informados os dados do beneficiário e informações do pagamento conforme os exemplos abaixo:
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

  /v1/pix/payments

HTTP Request Body - DICT

  {
    "initiation_type": "dict", 
    "idempotent_id": "PAGAMENTO_PARCEIRO01", 
    "receiver_name": "John Doe", 
    "receiver_document": "67178678097", 
    "value_cents": 10000,
    "pix_key_type": "cpf",
    "pix_key": "67178678097",
    "authorized": false
  }

HTTP Request Body - MANUAL

  {
    "initiation_type": "manual", 
    "idempotent_id": "PAGAMENTO_PARCEIRO02", 
    "value_cents": 10000,
    "receiver_name": "John Doe", 
    "receiver_document": "67178678097", 
    "receiver_bank_ispb": "00000000", 
    "receiver_agency": "0000", 
    "receiver_account": "00000000", 
    "receiver_account_type": "CACC", 
    "authorized": false
  }

Descrição dos atributos
ATRIBUTOS	DESCRIÇÃO	TIPO
initiation_type
(Opcional)	Define o tipo de pagamento que será realizado podendo ser DICT (utiliando chave Pix) ou Manual (utilizando dados da conta bancária)	ENUM
dict (Chave Pix)
manual (Dados conta bancária)
idempotent_id*
(Obrigatório)	Identificador único do pagamento no parceiro, a fim de evitar solicitações duplicadas	STRING
Limite de 255 caracteres
receiver_name
(Opcional)	Nome do beneficiário
Obrigatório caso initiation_type = manual	STRING
Limite de 100 caracteres
receiver_document
(Opcional)	Documento (CPF/CNPJ) do beneficiário
Obrigatório caso initiation_type = manual	STRING
Limite de 14 caracteres apenas números
receiver_bank_ispb
(Opcional)	Código ISPB identificador do banco da conta do beneficiário
Obrigatório caso initiation_type = manual	STRING
Limite de 20 caracteres apenas números
receiver_agency
(Opcional)	Agência da conta do beneficiário
Obrigatório caso initiation_type = manual	STRING
Limite de 10 caracteres apenas números
receiver_account
(Opcional)	Número da conta do beneficiário, com dígito verificador
Obrigatório caso initiation_type = manual	STRING
Limite de 20 caracteres apenas números
receiver_account_type
(Opcional)	Tipo da conta do beneficiário
Obrigatório caso initiation_type = manual	ENUM
CACC (Conta corrente)
SVGS (Conta poupança)
SLRY (Conta salário)
TRAN (Conta de pagamento)
value_cents
(Obrigatório)	Valor, em centavos, da cobrança	INTEGER
Maior que 0
pix_key_type
(Opcional)	Tipo de chave Pix do beneficiário
Obrigatório caso initiation_type = dict	ENUM
phone (Telefone)
email (Email)
cpf (CPF)
cnpj (CNPJ)
token (Chave Aleatória)
pix_key
(Opcional)	Chave Pix do beneficiário
Obrigatório caso initiation_type = dict	STRING
O formato de uma chave Pix varia de acordo com o tipo da chave:

Telefone: +5500000000009.
E-mail: john_doe@example.com.
CPF: 00000000000 (11)
CNPJ: 00000000000000 (14)
Chave aleatória: 123e4567- e12b-12d1-a456- 426655440000 (36)
authorized
(Opcional)	Cadastrar pagamento com autorização automática.	BOOLEAN
Se não informado, padrão: false
Sucesso

Após a chamada, é retornado um JSON com o status 201 - Created caso a operação tenha sido aceita pela API.
HTTP 201 Response Body - Exemplo

  {
    "payment": {
      "reference_code": "TEST01", 
      "idempotent_id": "PAGAMENTO_PARCEIRO01", 
      "value_cents": 10000,
      "pix_key_type": "cpf",
      "pix_key": "67178678097", 
      "receiver_name": "John Doe", 
      "receiver_document": "67178678097", 
      "status": "authorization_pending"
    } 
  }

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
payment
(Obrigatório)	Objeto de dados do qrcode cadastrado.	OBJECT
payment.idempotent_id
(Obrigatório)	Identificador único do pagamento no parceiro	STRING
Limite de 255 caracteres
payment.receiver_name
(Obrigatório)	Nome do beneficiário	STRING
Limite de 100 caracteres
payment.receiver_document
(Obrigatório)	Documento (CPF/CNPJ) do beneficiário	STRING
Limite de 14 caracteres
payment.value_cents
(Obrigatório)	Valor, em centavos, do pagamento.	NTEGER
Maior que zero
payment.pix_key_type
(Obrigatório)	Tipo de chave Pix do beneficiário	ENUM
phone (Telefone)
email (Email)
cpf (CPF)
cnpj (CNPJ)
token (Chave Aleatória)
payment.pix_key
(Obrigatório)	Chave Pix do beneficiário	STRING
O formato de uma chave Pix varia de acordo com o tipo da chave:

Telefone: +5500000000009
E-mail: john_doe@example.com
CPF: 00000000000 (11) CNPJ: 00000000000000 (14)
Chave aleatória: 123e4567-e12b-12d1- a456-426655440000 (36)
payment.reference_code
(Obrigatório)	Identificador interno do pagamento	STRING
Limite de 100 caracteres
payment.status
(Obrigatório)	Status do pagamento	ENUM
authorization_pending (Autorização pendente)
**auto_authorization **(Enviado para autorização automática)
canceled (Cancelado)
sent (Aguardando confirmação)
completed (Finalizado)
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

  {
    "error": "pix_key required"
  }

HTTP 422 Response Body - Exemplo

  {
    "error": " Insufficient funds"
  }

HTTP 403 Response Body - Exemplo

  {
    "error": "Receiver not allowed"
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
Listar Pagamentos

Permite obter uma lista de pagamentos cadastrados atribuindo diversos filtros para selecionar os itens desejados. A resposta vem em formato paginado, sendo no máximo 100 registros por página. Um atributo meta será retornado informando os parâmetros totalizadores e o total de páginas da consulta realizada.
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  /v1/pix/payments

URL Params – Filter Options
PARÂMETRO	DESCRIÇÃO	TIPO
page
(Opcional)	Número da página. Se não informado, assume o valor 1	INTEGER
maior que 0
registration_start_date
(Obrigatório caso registration_end_date seja informado)	Valor inicial para o período de busca por data de geração das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
registration_end_date
(Obrigatório caso registration_start_date seja informado)	Valor final para o período de busca por data geração das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
payment_start_date
(Obrigatório caso payment_end_date seja informado)	Valor inicial para o período de busca por data de pagamento das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
payment_end_date
(Obrigatório caso payment_start_date seja informado)	Valor final para o período de busca por data pagamento das cobranças	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
receiver_name
(Opcional)	Nome do beneficiário. Consulta do tipo LIKE	STRING
limite 100 caracteres
receiver_document
(Opcional)	CPF/CNPJ do beneficiário. Consulta do tipo LIKE	STRING
limite 14 caracteres apenas números
status
(Opcional)	Status das cobranças	STRING (Enum)
error (Erro ao gerar cobrança)
awaiting_payment (Aguardando Pagamento)
paid (Pago)
canceled (Cancelado)
reference_code
(Opcional)	Código de identificação. Consulta do tipo LIKE	STRING
limite 100 caracteres
end_to_end
(Opcional)	Código de pagamento. Consulta do tipo LIKE	STRING
limite 32 caracteres
Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

{
    "payments": [
        {
        "reference_code": "TEST01",
        "idempotent_id": "PAGAMENTO_PARCEIRO01", 
        "value_cents": 10000,
        "pix_key_type": "cpf",
        "pix_key": "67178678097",
        "receiver_name": "John Doe", 
        "receiver_document": "67178678097",
        "status": "completed",
        "registration_date": "2021-11-10T14:51:25.000-03:00", 
        "payment_date": "2021-11-10T14:52:10.000-03:00", 
        "cancellation_date": null,
        "cancellation_reason": null,
        "end_to_end": "E18236120202206142202a1022c1tg10"
        } 
    ],
    "meta": {
        "current_page": 1, 
        "total_pages": 1, 
        "total_items_amount": 1, 
        "total_value_cents": 2
    } 
    }

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
payments
(Obrigatório)	Lista de pagamentos Pix que obedecem os filtros especificados	ARRAY
payments.reference_code
(Obrigatório)	Identificador único do pagamento	STRING
Limite de 100 caracteres
payments.idempotent_id
(Obrigatório)	Identificador único do pagamento no parceiro	STRING
Limite de 255 caracteres
payments.value_cents
(Obrigatório)	Valor, em centavos, do pagamento.	NTEGER
Maior que zero
payments.pix_key_type
(Obrigatório)	Tipo de chave PIX do beneficiário	ENUM
phone (Telefone)
email (Email)
cpf (CPF)
cnpj (CNPJ)
token (Chave Aleatória)
payments.pix_key
(Obrigatório)	Chave Pix do beneficiário	STRING
O formato de uma chave Pix varia de acordo com o tipo da chave:

Telefone: +5500000000009
E-mail: john_doe@example.com
CPF: 00000000000 (11) CNPJ: 00000000000000 (14)
Chave aleatória: 123e4567-e12b-12d1- a456-426655440000 (36)
payments.receiver_name
(Obrigatório)	Nome do beneficiário	STRING
Limite de 100 caracteres
payments.receiver_document
(Obrigatório)	Documento (CPF/CNPJ) do beneficiário	STRING
Limite de 14 caracteres
payments.status
(Obrigatório)	Status do pagamento	ENUM
authorization_pending (Autorização pendente)
**auto_authorization **(Enviado para autorização automática)
canceled (Cancelado)
sent (Aguardando confirmação)
completed (Finalizado)
payments.registration_date
(Obrigatório)	Data de criação do pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payments.payment_date
(Obrigatório)	Data de pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payments.cancellation_date
(Obrigatório)	Data de cancelamento do pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payments.cancellation_reason
(Opcional)	Motivo de cancelamento do pagamento	STRING
Limite de 255 caracteres
payments.end_to_end
(Opcional)	Identificador único do pagamento Pix	STRING
Limite de 32 caracteres
meta
(Obrigatório)	Objeto que contém os metadados da requisição	OBJECT
meta.current_page
(Obrigatório)	Página atual da consulta	INTEGER
maior que 0
meta.total_pages
(Obrigatório)	Total de páginas da consulta realizada	INTEGER
maior que 0
meta.total_items_amount
(Obrigatório)	Quantidade total de cobranças existentes considerando todas as páginas	INTEGER
maior que 0
meta.total_value_cents
(Obrigatório)	Valor total, em centavos das cobranças considerando todas as páginas	INTEGER
maior que 0
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

{
  "error": "registration_end_date required"
}

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
Consultar Pagamento - Código Referência

Retorna de forma mais detalhada os dados de um pagamento Pix. Para fazê-lo deve ser efetuada a chamada para o serviço de consulta de pagamento, enviando como parâmetros o identificador único (reference_code).
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  v1/pix/payments/{reference_code}

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

  {
    "payment": {
      "reference_code": "TEST01",
      "idempotent_id": "PAGAMENTO_PARCEIRO01", 
      "value_cents": 10000,
      "pix_key": "67178678097",
      "pix_key_type": "cpf",
      "receiver_name": "Joe Doe",
      "receiver_document": "67178678097", 
      "receiver_bank_name": "Nome Banco S.A.", 
      "receiver_agency": "000001",
      "receiver_account": "2398438", 
      "receiver_account_type": "checking_account",
      "status": "completed",
      "registration_date": "2021-07-16T08:14:04.000-03:00", 
      "payment_date": "2021-07-16T08:14:04.000-03:00", 
      "cancellation_date": null,
      "cancellation_reason": null,
      "end_to_end ": "E18236120202206142202a1022c1tg10",
    }
  }

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
payment
(Obrigatório)	Objeto de dados do qrcode cadastrado.	OBJECT
payment.idempotent_id
(Obrigatório)	Identificador único do pagamento no parceiro	STRING
Limite de 255 caracteres
payment.reference_code
(Obrigatório)	Identificador único do pagamento	STRING
Limite de 100 caracteres
payment.value_cents
(Obrigatório)	Valor, em centavos, do pagamento.	NTEGER
Maior que zero
payment.pix_key_type
(Obrigatório)	Tipo de chave Pix do beneficiário	ENUM
phone (Telefone)
email (Email)
cpf (CPF)
cnpj (CNPJ)
token (Chave Aleatória)
payment.pix_key
(Obrigatório)	Chave Pix do beneficiário	STRING
O formato de uma chave Pix varia de acordo com o tipo da chave:

Telefone: +5500000000009
E-mail: john_doe@example.com
CPF: 00000000000 (11) CNPJ: 00000000000000 (14)
Chave aleatória: 123e4567-e12b-12d1- a456-426655440000 (36)
payment.receiver_name
(Obrigatório)	Nome do beneficiário	STRING
Limite de 100 caracteres
payment.receiver_document
(Obrigatório)	Documento (CPF/CNPJ) do beneficiário	STRING
Limite de 14 caracteres
payment.receiver_bank_name
(Opcional)	Nome da instituição financeira do beneficiário	STRING
Limite de 50 caracteres
payment.receiver_agency
(Opcional)	Agência da conta bancária do beneficiário	STRING
Limite de 20 caracteres
payment.receiver_account
(Opcional)	Número da conta bancária do beneficiário	STRING
Limite de 20 caracteres
payment.receiver_account_type
(Opcional)	Tipo de conta bancária do beneficiário	ENUM
checking_account
savings_account
salary_account
payment.status
(Obrigatório)	Status do pagamento	ENUM
authorization_pending (Autorização pendente)
**auto_authorization **(Enviado para autorização automática)
canceled (Cancelado)
sent (Aguardando confirmação)
completed (Finalizado)
payment.registration_date
(Obrigatório)	Data de criação do pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payment.payment_date
(Obrigatório)	Data de pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payment.cancellation_date
(Obrigatório)	Data de cancelamento do pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payment.cancellation_reason
(Opcional)	Motivo de cancelamento do pagamento	STRING
Limite de 255 caracteres
payment.end_to_end
(Opcional)	Identificador único do pagamento Pix	STRING
Limite de 32 caracteres
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 406 Response Body - Exemplo:

  {
    "error": "Payment not found."
  }

HTTP 500 Response Body - Exemplo:

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
Consultar Pagamento - Idempotente

Similar ao endpoint de consulta de pagamento por código de referência, desta vez sendo necessário informar como parâmetro o id idempotente(idempotent_id).
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  v1/pix/payments/{idempotent_id}/idempotent_id

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

  {
    "payment": {
      "reference_code": "TEST01",
      "idempotent_id": "PAGAMENTO_PARCEIRO01", 
      "value_cents": 10000,
      "pix_key": "67178678097",
      "pix_key_type": "cpf",
      "receiver_name": "Joe Doe",
      "receiver_document": "67178678097", 
      "receiver_bank_name": "Nome Banco S.A.", 
      "receiver_agency": "000001",
      "receiver_account": "2398438", 
      "receiver_account_type": "checking_account",
      "status": "completed",
      "registration_date": "2021-07-16T08:14:04.000-03:00", 
      "payment_date": "2021-07-16T08:14:04.000-03:00", 
      "cancellation_date": null,
      "cancellation_reason": null,
      "end_to_end ": "E18236120202206142202a1022c1tg10",
    }
  }

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
payment
(Obrigatório)	Objeto de dados do qrcode cadastrado.	OBJECT
payment.idempotent_id
(Obrigatório)	Identificador único do pagamento no parceiro	STRING
Limite de 255 caracteres
payment.reference_code
(Obrigatório)	Identificador único do pagamento	STRING
Limite de 100 caracteres
payment.value_cents
(Obrigatório)	Valor, em centavos, do pagamento.	NTEGER
Maior que zero
payment.pix_key_type
(Obrigatório)	Tipo de chave Pix do beneficiário	ENUM
phone (Telefone)
email (Email)
cpf (CPF)
cnpj (CNPJ)
token (Chave Aleatória)
payment.pix_key
(Obrigatório)	Chave Pix do beneficiário	STRING
O formato de uma chave Pix varia de acordo com o tipo da chave:

Telefone: +5500000000009
E-mail: john_doe@example.com
CPF: 00000000000 (11) CNPJ: 00000000000000 (14)
Chave aleatória: 123e4567-e12b-12d1- a456-426655440000 (36)
payment.receiver_name
(Obrigatório)	Nome do beneficiário	STRING
Limite de 100 caracteres
payment.receiver_document
(Obrigatório)	Documento (CPF/CNPJ) do beneficiário	STRING
Limite de 14 caracteres
payment.receiver_bank_name
(Opcional)	Nome da instituição financeira do beneficiário	STRING
Limite de 50 caracteres
payment.receiver_agency
(Opcional)	Agência da conta bancária do beneficiário	STRING
Limite de 20 caracteres
payment.receiver_account
(Opcional)	Número da conta bancária do beneficiário	STRING
Limite de 20 caracteres
payment.receiver_account_type
(Opcional)	Tipo de conta bancária do beneficiário	ENUM
checking_account
savings_account
salary_account
payment.status
(Obrigatório)	Status do pagamento	ENUM
authorization_pending (Autorização pendente)
**auto_authorization **(Enviado para autorização automática)
canceled (Cancelado)
sent (Aguardando confirmação)
completed (Finalizado)
payment.registration_date
(Obrigatório)	Data de criação do pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payment.payment_date
(Obrigatório)	Data de pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payment.cancellation_date
(Obrigatório)	Data de cancelamento do pagamento	STRING
Formato datetime YYYY-mm-ddTHH:MM:ss.z
payment.cancellation_reason
(Opcional)	Motivo de cancelamento do pagamento	STRING
Limite de 255 caracteres
payment.end_to_end
(Opcional)	Identificador único do pagamento Pix	STRING
Limite de 32 caracteres
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 406 Response Body - Exemplo:

  {
    "error": "Payment not found."
  }

HTTP 500 Response Body - Exemplo:

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
Listar Tipos de Webhooks

A API de Webhooks permite que o parceiro configure como receberá as mensagens de notificação originadas pela API Pix. É possível definir uma URL para cada tipo de mensagem e um código de autorização que será enviado como Header Authorization, garantindo assim que a mensagem recebida teve origem na Zendry. Se o parceiro achar necessário, poderá também validar a requisição pelo endereço IP de origem, que poderá ser obtido após solicitação ao atendimento Zendry.
Fazendo Requisição

Retorna uma lista de tipos de notificações existentes. Necessária para utilizar corretamente os endpoints Cadastrar/Alterar Webhook e Remover Webhook.

A chamada deverá ser feita utilizando o método GET.
URL

   /v1/webhooks/types

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

  {
    "webhook_types": [
      {
        "id": 1,
        "name": "pix_qrcodes"
      },
      {
        "id": 2,
        "name": "pix_payments"
      },
    ] 
  }

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
webhooks_types
(Obrigatório)	Lista de tipos de webhooks	ARRAY
webhook_types.id
(Obrigatório)	Identificador único do tipo de webhook	NUMBER
webhook_types.name
(Obrigatório)	Nome do tipo de webhook	STRING
Limite de 255 caracteres

Cadastrar/Alterar Webhook

Cadastra novo webhook para recebimento de mensagens de um determinado tipo. Caso já exista uma configuração anterior, os dados serão sobrescritos.
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

   /v1/webhooks/{webhook_type_id}

HTTP Request Body

  {
    "url": "https://test2.com",
    "authorization": "authorization_value"
  }

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
url
(Obrigatório)	Endereço destino da mensagem do webhook	STRING
Limite de 255 caracteres, iniciando com https://
authorization
(Opcional)	Valor que será enviado no header Authorization para comprovar que é a Zendry que está enviando a mensagem	STRING
Limite de 100 caracteres
Sucesso

Em caso de sucesso, será retornado uma mensagem informando o êxito da operação com código HTTP 201 - Created em caso de cadastro de uma nova configuração de webhook, e HTTP 200 – OK para atualização dos dados.
HTTP 200 Response Body - Exemplo

  {
    "message": "Operation succeeded" 
  }
  Remover Webhook

Remove uma configuração de webhook previamente estabelecida. Isso significa interromper o envio de mensagens de um determinado tipo para o parceiro, uma vez que não existirão configurações para definir como esses dados seriam enviados.
Fazendo Requisição

A chamada deverá ser feita utilizando o método DELETE.
URL

    /v1/webhooks/{webhook_type_id}

Sucesso

Em caso de sucesso, será retornado uma mensagem informando o êxito da operação com código HTTP 201 em caso de cadastro de uma nova configuração de webhook, e HTTP 200 – OK para atualização dos dados.
HTTP 200 Response Body - Exemplo

  {
    "message": "Operation succeeded" 
  }
  
  Notificação QRCodes

Quando um QRCode for pago, o sistema enviará uma notificação para o endereço fornecido pelo cliente, informando atualização de status do mesmo.

A URL que receberá as notificações deverá ser informada através do endpoint Cadastrar/Alterar Webhook.
Fazendo Requisição

A notificação será enviada utilizando o método POST, e espera uma resposta do tipo HTTP 200.

Segue estrutura do JSON enviado como request body:
HTTP Request Body - PIX QRCODE

    {
        "notification_type": "pix_qrcode",
        "message": {
            "reference_code": "ZENDRYPIXQRCODE2",
            "value_cents": 2,
            "content": "00020101021126580014br.gov.bcb.pix0136d5091c68-5056-481b-88ad-95eb340a1a2152040000530398654040.025802BR5925Zendry Solucoes em Paga6009SAO PAULO62220518ZENDRYPIXQRCODE263044FC9",
            "status": "paid",
            "generator_name": "John Doe",
            "generator_document": "67178678097",
            "payer_name": "John Doe",
            "payer_document": "67178678097", 
            "registration_date": "2021-11-10T14:51:25.000-03:00", 
            "payment_date": "2021-11-10T14:52:10.000-03:00",
            "end_to_end": "E18236120202206142202a1022c1tg10"
        }, 
        "md5":"679f3ff14b8eadd1e504f2a35c0d8fb3"
    }

HTTP Request Body - PIX STATIC QRCODE

    {
        "notification_type": "pix_static_qrcode",
        "message": {
            "reference_code": "ZENDRYPIXQRCODE2",
            "value_cents": 2,
            "content": "00020101021126580014br.gov.bcb.pix0136d5091c68-5056-481b-88ad-95eb340a1a2152040000530398654040.025802BR5925Zendry Solucoes em Paga6009SAO PAULO62220518ZENDRYPIXQRCODE263044FC9",
            "status": "paid",
            "generator_name": "John Doe",
            "generator_document": "67178678097",
            "payer_name": "John Doe",
            "payer_document": "67178678097", 
            "registration_date": "2021-11-10T14:51:25.000-03:00", 
            "payment_date": "2021-11-10T14:52:10.000-03:00",
            "end_to_end": "E18236120202206142202a1022c1tg10"
        }, 
        "md5":"679f3ff14b8eadd1e504f2a35c0d8fb3"
    }

Erro

Caso seja obtido um resultado diferente de 200, o sistema tentará enviar pelos próximos 10 minutos, a cada minuto ou enquanto não for obtido um resultado de sucesso.

O erro persistindo nas 10 tentativas, a notificação será marcada como cancelada no sistema e nossa equipe entrará em contato para averiguar quaisquer problemas de integração.

OBS: Para gerar o hash md5 da mensagem é necessário considerar o seguinte formato da STRING a ser codificada.
STRING

    qrcode.{reference_code}.{end_to_end}.{value_cents}.{secret_key}

Onde:

    qrcode: palavra qrcode escrita em minúsculo
    {reference_code}* Código de referência qrcode Pix
    {end_to_end}: Identificador único do pagamento Pix
    {value_cents}: Valor em centavos
    {secret_key}: Chave única, exclusiva do cliente para gerar hash. ela será gerada junto a credenciais de acesso da API

A string abaixo deve ser montada para gerar o Hash MD5 da mensagem citada como exemplo acima, considerando que a chave secreta do cliente seja a palavra SECRETKEY:
STRING - Exemplo

    qrcode.ZENDRYPIXQRCODE2.E18236120202206142202a1022c1tg10.2.SECRETKEY

Observação

O reenvio de notificações após a primeira tentativa falha ocorrerá apenas em ambiente de produção.

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
notification_type
(Obrigatório)	Indica o tipo de mensagem transmitida	ENUM
pix_qrcode
(QRcode pix dinâmico)
pix_static_qrcode
(QRcode pix estático)
message
(Obrigatório)	Objeto qrcode atualizado, conforme apresentado pelo endpoint Listar Qrcodes	OBJECT
message.reference_code
(Obrigatório)	Código de identificação	STRING
limite 100 caracteres
message.value_centes
(Obrigatório)	Valor da cobrança, em centavos	INTEGER
Maior que 0
message.content
(Obrigatório)	Conteúdo do QRCode. (Código copia e cola Pix)	STRING
limite de 255 caracteres
message.status
(Obrigatório)	Status das cobranças	STRING (Enum)
error (Erro ao gerar cobrança)
awaiting_payment (Aguardando Pagamento)
paid (Pago)
canceled (Cancelado)
message.generator_name
(Obrigatório)	Nome do gerador das cobrança.	STRING
limite 100 caracteres apenas números
message.generator_document
(Obrigatório)	CPF/CNPJ do gerador das cobrança.	STRING
limite 14 caracteres apenas números
message.payer_name
(Obrigatório)	Nome do pagador das cobrança.	STRING
limite 100 caracteres apenas números
message.payment_document
(Obrigatório)	CPF/CNPJ do pagador das cobrança.	STRING
limite 14 caracteres apenas números
message.registration_date
(Obrigatório)	Valor da data de geração da cobrança.	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
message.payment_date
(Obrigatório)	Valor da data de pagamento da cobrança.	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
message.end_toend
(Obrigatório)	Identificador único do pagamento Pix	STRING
limite de 32 caracteres
md5
(Opcional)	Hash md5 para autenticação da notificação	STRING
Notificação Pagamentos

Quando houver alguma alteração de status em um pagamento, notificação para o webhook cadastrado pelo cliente, via API, informando a atualização de valores.
Fazendo Requisição

A notificação será enviada utilizando o método POST, e espera uma resposta do tipo HTTP 200.

Segue estrutura do JSON enviado como request body:
HTTP Request Body

    {
        "notification_type": "pix_payment",
        "message": {
            "value_cents": 1,
            "reference_code": "P202306150000039870", 
            "idempotent_id": "TEST300523-05",
            "pix_key": "09354532454",
            "pix_key_type": "cpf",
            "status": "completed",
            "end_to_end": null,
            "receiver_name": "Usuário CREDCEOS", 
            "receiver_document": "00000000000", 
            "registration_date": "2023-06-15T09:49:52.000-03:00", 
            "payment_date": "2023-06-15T09:49:54.000-03:00", 
            "cancellation_date": null,
            "cancellation_reason": null,
            "receipt_url": null
        },
        "md5": "9b5b3e1e7659d81c406bd40a96c83294"
    }

Erro

Caso seja obtido um resultado diferente de 200, o sistema tentará enviar pelos próximos 10 minutos, a cada minuto ou enquanto não for obtido um resultado de sucesso.

O erro persistindo nas 10 tentativas, a notificação será marcada como cancelada no sistema e nossa equipe entrará em contato para averiguar quaisquer problemas de integração.

OBS: Para gerar o hash md5 da mensagem é necessário considerar o seguinte formato da STRING a ser codificada.
STRING

    payment.{reference_code}.{idempotent_id}.{value_cents}.{secret_key}

Onde:

    payment: palavra payment escrita em minúsculo
    {reference_code}: Código de referência qrcode Pix
    {idempotent_id}: Identificador único do pagamento no parceiro, a fim de evitar solicitações duplicadas
    {value_cents}: Valor em centavos
    {secret_key}: Chave única, exclusiva do cliente para gerar hash. ela será gerada junto a credenciais de acesso da API

A string abaixo deve ser montada para gerar o Hash MD5 da mensagem citada como exemplo acima, considerando que a chave secreta do cliente seja a palavra SECRETKEY:
STRING - Exemplo

    payment.P202306150000039870.TEST300523-05.1.SECRETKEY

Observação

O reenvio de notificações após a primeira tentativa falha ocorrerá apenas em ambiente de produção.

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
notification_type
(Obrigatório)	Indica o tipo de mensagem transmitida	STRING
Limite 25 caracteres
message
(Obrigatório)	Objeto qrcode atualizado, conforme apresentado pelo endpoint Listar Pagamentos	OBJECT
md5
(Opcional)	Hash md5 para autenticação da notificação	STRING
Notificação Crypto

Quando um Criptomoeda for pago, o sistema enviará uma notificação para o endereço fornecido pelo cliente, informando atualização de status do mesmo.

A URL que receberá as notificações deverá ser informada através do endpoint Cadastrar/Alterar Webhook.
Recebimento de Criptomoeda
Fazendo Requisição

A notificação será enviada utilizando o método POST, e espera uma resposta do tipo HTTP 200.

Segue estrutura do JSON enviado como request body:
HTTP Request Body

    {
        "notification_type":"crypto_receivement",
        "message":{
            "value":10.5,
            "wallet_id":"19114010-0487-4d7d-b15f-efb6213de9ea",
            "payer_address":"TOSJDFSOIDFJSOFISDF",
            "payment_date":"2024-08-13T11:10:02.965-03:00",
            "operation_code":"dc41da0b-9dd4-49e9-bec2-5b34a68556f5"
        }
    }

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
notification_type
(Obrigatório)	Indica o tipo de mensagem transmitida.	ENUM
crypto_receivement
message
(Obrigatório)	Atualizado, conforme apresentado pelo endpoint.	STRING
limite de 255 caracteres
message.value
(Obrigatório)	Valor, em centavos.	DECIMAL
Maior que 0
message.wallet_id
(Obrigatório)	ID da carteira cadastrada.	STRING
message.payer_address
(Obrigatório)	Endereço do destinatário que receberá as moedas	STRING
message.payment_date
(Obrigatório)	Data do pagamento.	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
message.operation_code
(Obrigatório)	Código da operação.	STRING
limite de 255 caracteres
Erro

Caso seja obtido um resultado diferente de 200, o sistema tentará enviar pelos próximos 10 minutos, a cada minuto ou enquanto não for obtido um resultado de sucesso.

O erro persistindo nas 10 tentativas, a notificação será marcada como cancelada no sistema e nossa equipe entrará em contato para averiguar quaisquer problemas de integração.

OBS: Para gerar o hash md5 da mensagem é necessário considerar o seguinte formato da STRING a ser codificada.
STRING

    cryptoreceivement.{address}.{operation_code}.{value}.{secret_key}

Onde:

    cryptoreceivement: A palavra cryptoreceivement escrita em minúsculo
    {address} Endereço do destinatário que receberá as moedas
    {operation_code}: Código da operação
    {value}: Valor em centavos
    {secret_key}: Chave única, exclusiva do cliente para gerar hash. ela será gerada junto a credenciais de acesso da API

A string abaixo deve ser montada para gerar o Hash MD5 da mensagem citada como exemplo acima, considerando que a chave secreta do cliente seja a palavra SECRETKEY:
STRING - Exemplo

    cryptoreceivement.address.operation_code.value.secret_key

Observação

O reenvio de notificações após a primeira tentativa falha ocorrerá apenas em ambiente de produção.
Pagamento em Criptomoeda
Fazendo Requisição

A notificação será enviada utilizando o método POST, e espera uma resposta do tipo HTTP 200.

Segue estrutura do JSON enviado como request body:
HTTP Request Body

{
   "notification_type":"crypto_payment",
   "message":{
      "crypto_currency_code":"USDT",
      "reference_code":"CE202408130000000042",
      "receiver_address":"TLyxu5on2Jdkcn5Y9TBfoTCXaQR4gG8rAL",
      "value":1.49,
      "status":"completed",
      "operation_code":"test42",
      "payment_date":"2024-08-13T11:19:51.945-03:00",
      "return_date":"2024-08-13T11:19:51.945-03:00",
      "return_message":null
   }
}

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
notification_type
(Obrigatório)	Indica o tipo de mensagem transmitida.	ENUM
crypto_payment
message
(Obrigatório)	Atualizado, conforme apresentado pelo endpoint.	STRING
limite de 255 caracteres
message.crypto_currency_code
(Obrigatório)	código cripto.	STRING
message.reference_code
(Obrigatório)	código de referência.	STRING
message.receiver_address
(Obrigatório)	Endereço do recebedor.	STRING
message.value
(Obrigatório)	Valor, em centavos.	DECIMAL
Maior que 0
message.status
(Obrigatório)	Status do pagamento.	ENUM
message.operation_code
(Obrigatório)	Código da operação.	STRING
limite de 255 caracteres
message.wallet_id
(Obrigatório)	ID da carteira cadastrada.	STRING
message.payer_address
(Obrigatório)	Endereço do pagador	STRING
message.payment_date
(Obrigatório)	Data do pagamento.	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
message.return_date
(Obrigatório)	Data do retorno do pagamento.	STRING
formato datetime YYYY-mm-ddTHH:MM:ss
Erro

Caso seja obtido um resultado diferente de 200, o sistema tentará enviar pelos próximos 10 minutos, a cada minuto ou enquanto não for obtido um resultado de sucesso.

O erro persistindo nas 10 tentativas, a notificação será marcada como cancelada no sistema e nossa equipe entrará em contato para averiguar quaisquer problemas de integração.

OBS: Para gerar o hash md5 da mensagem é necessário considerar o seguinte formato da STRING a ser codificada.
STRING

    cryptopayment.{reference_code}.{confirmation_code}.{value}.{secret_key}"

Onde:

    cryptopayment: A palavra cryptoreceivement escrita em minúsculo
    {reference_code} Código de referência
    {confirmation_code}: Código da operação
    {value}: Valor em centavos
    {secret_key}: Chave única, exclusiva do cliente para gerar hash. ela será gerada junto a credenciais de acesso da API

A string abaixo deve ser montada para gerar o Hash MD5 da mensagem citada como exemplo acima, considerando que a chave secreta do cliente seja a palavra SECRETKEY:
STRING - Exemplo

    cryptoreceivement.ZENDRYPIXQRCODE2.E18236120202206142202a1022c1tg10.2.SECRETKEY

Observação

O reenvio de notificações após a primeira tentativa falha ocorrerá apenas em ambiente de produção.Listar Webhooks

Permite que o parceiro liste os endereços webhooks cadastrados.
Fazendo requisição

Retorna uma lista de endereços webhooks cadastrados.

A chamada deverá ser feita utilizando o método GET.
URL

   /v1/webhooks

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
Listar Webhooks

{
    "webhooks": [
        {
            "url": "https://yourdomain.com/api-payments/webhook.php",
            "webhook_type_id": 1,
            "authorization_header": false
        },
        {
            "url": "https://yourdomain.com/pix_payments_receiver",
            "webhook_type_id": 2,
            "authorization_header": true
        }
    ]
}

Descrição dos Atributos
PARÂMETRO	DESCRIÇÃO	TIPO
webhooks
(Obrigatório)	Lista de webhooks cadastrados	ARRAY
webhooks.url
(Obrigatório)	Endereço destino da mensagem do webhook	STRING
Limite de 255 caracteres, iniciando com https://
webhooks.webhook_type_id
(Obrigatório)	Define o tipo de webhook cadastrado	NUMBER
1 (pix_qrcodes)
2 (pix_payments)
webhooks.authorization_header
(Obrigatório)	Indica se o webhook cadastro necessita do envio do authorization_code nas requisições	BOOLEAN
Recuperar saldo da conta

Recupera informações de saldo da conta
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  /v1/account/balance

Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

{
  "data": {
    "account_balance": {
      "available_value_cents": 0,
      "blocked_value_cents": 0,
      "total_value_cents": 0
    },
    "status": "string"
  }
}

Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 401 Response Body - Exemplo

 {
  "data": {
    "error": "string",
    "details": "string"
  },
  "status": "string"
}
Gerar Pagamento

Gera um pagamento em criptomoeda USDT.
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

  /v1/crypto/payments

HTTP Request Body

{
    "value": 1.47,
    "crypto_currency": "usdt",
    "receiver_address": "TLyxu5on2Jdkcn5Y9TBfoTCXaQR4gG8rAL",
    "sender_wallet_id": "39dd6455-f071-4922-a4ed-c6f3e879fb12",
}

Descrição dos atributos
ATRIBUTOS	DESCRIÇÃO	TIPO
value
(Obrigatório)	Valor a ser transferido	DECIMAL
maior que 0
crypto_currency
(Obrigatório)	Código da criptomoeda	STRING
Disponível apenas (usdt)
receiver_address
(Obrigatório)	Endereço do destinatário que receberá as moedas	STRING
sender_wallet_id
(Obrigatório)	ID da carteira do remetente cadastrada previamente na Zendry através do endpoint Gerar Carteira	STRING
Sucesso

Após a chamada, é retornado um JSON com o status 201 - Created caso o procedimento tenha ocorrido com sucesso.
HTTP 201 Response Body - Exemplo

{
    "crypto_payment": {
        "value": "1.49",
        "crypto_currency": "usdt",
        "reference_code": "CE202408180000000057",
        "status": "pending"
    }
}

Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

  {
    "msg": "Invalid crypto currency"
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
  Gerar Carteira

Gera uma nova carteira de criptomoedas.
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

  /v1/crypto/wallets

Sucesso

Após a chamada, é retornado um JSON com o status 201 - Created caso o procedimento tenha ocorrido com sucesso.
HTTP 201 Response Body - Exemplo

{
    "crypto_wallet": {
        "wallet_id": "e16cc838-ac51-4012-8d7a-697cb0bfa16d",
        "address": "TLxqaPTgvtUHqqvMmCjGBgVUmpckuvgN2r"
    }
}

Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

  {
    "msg": "Operation failed! Please try again or contact our support team"
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
  Simular Recebimento

Simula o recebimento de criptomoeda (Funciona apenas em homologação).
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

  /v1/crypto/receivement

HTTP Request Body

{
    "value": 10.50,
    "receiver_wallet_address": "TJcLHew3otEuCaBJzqqN82V2Y2Bdqfyx7t",
    "sender_wallet_address": "TOSJDFSOsdfswwerwerfIDFJSOFISDF"
}

Descrição dos atributos
ATRIBUTOS	DESCRIÇÃO	TIPO
value
(Obrigatório)	Valor a ser recebido	DECIMAL
maior que 0
receiver_wallet_address
(Obrigatório)	Endereço da carteira do destinatário (deve estar cadastrado na Zendry pelo endpoint Gerar Carteira)	STRING
sender_wallet_address
(Obrigatório)	Endereço da carteira do remetente	STRING
Sucesso

Após a chamada, é retornado um JSON com o status 202 - Accepted caso o procedimento tenha ocorrido com sucesso.
Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

  {
    "msg": ""
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
Obter Cotação USDT

Obtém a cotação atual da criptomoeda USDT.
Fazendo Requisição

A chamada deverá ser feita utilizando o método GET.
URL

  /v1/crypto/quotations/usdt

Sucesso

Após a chamada, é retornado um JSON com o status 200 - OK caso o procedimento tenha ocorrido com sucesso.
HTTP 201 Response Body - Exemplo

{
    "brl_price": "5.51486348"
}

Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 400 Response Body - Exemplo

  {
    "msg": "Operation failed! Please try again or contact our support team"
  }

HTTP 500 Response Body - Exemplo

  {
    "error": "Operation failed! Please try again or contact our support team"
  }
  Gerar checkout

Implemente um checkout eficiente para suas vendas
Fazendo Requisição

A chamada deverá ser feita utilizando o método POST.
URL

/v1/charges POST

HTTP Request Body

{
  "charge": {
    "value_cents": 0,
    "description": "string",
    "callback_url": "string",
    "expiration_date": "2024-10-23T17:42:09.477Z",
    "generator_document": "string",
    "generator_name": "string",
    "payment_methods": [
      "string"
    ]
  }
}

Descrição dos atributos
ATRIBUTOS	DESCRIÇÃO	TIPO
value_cents
(Obrigatório)	Valor a ser recebido	DECIMAL
maior que 0
description
(Obrigatório)	Descrição	STRING
callback_url	URL do webhook onde as notificações serão enviadas	STRING
expiration_date	Tempo de expiração do checkout	STRING formato datetime YYYY-mm-ddTHH:MM:ss
generator_document	Documento (CPF/CNPJ) do pagador, que será utilizado para identificação	STRING
generator_name	Nome do Pagador, que será utilizado para identificação	STRING
payment_methods
(Obrigatório)	Métodos de pagamento que será utilizado no checkout:
['pix', 'crypto']	array de strings.
Sucesso

Em caso de sucesso, será retornado uma mensagem HTTP 200 – OK, contendo os dados, conforme apresentado abaixo:
HTTP 200 Response Body - Exemplo

{
  "data": {
    "charge": {
      "reference_code": "string",
      "description": "string",
      "value_cents": 0,
      "created_at": "2024-10-23T17:42:09.478Z",
      "link": "string",
      "callback_url": "string",
      "payment_methods": [
        "string"
      ],
      "generated_payments": [
        {
          "id": 0,
          "ec": "string",
          "identification_code": "string",
          "value_cents": 0,
          "code": "string",
          "origin": "string",
          "status": "string",
          "payment_method": "string",
          "payment_date": "2024-10-23T17:42:09.478Z",
          "generator_name": "string",
          "generator_document": "string",
          "payer_name": "string",
          "payer_document": "string",
          "transaction_status": "string",
          "transaction_nsu": "string",
          "paid_value_cents": 0,
          "settlement_date": "2024-10-23T17:42:09.478Z",
          "created_at": "2024-10-23T17:42:09.478Z"
        }
      ]
    },
    "status": "string"
  }
}

Erros

Em caso de erros, será retornado um json com o atributo error especificando o motivo de a operação ter sido invalidada.
HTTP 422 Response Body - Exemplo

{
  "data": {
    "error": "string",
    "status": "string"
  }
}
