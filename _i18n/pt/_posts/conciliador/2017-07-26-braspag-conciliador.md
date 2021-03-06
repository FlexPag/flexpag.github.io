---
layout: manual
title: Manual de Arquivos
description: Integração Técnica
search: true
categories: manual
tags:
  - 6. Conciliador
language_tabs:
  xml: XML
---
# Agiliza Braspag

Para acessar a documentação oficial da API do Agiliza Braspag, [clique aqui](https://documenter.getpostman.com/view/2956778/T1Dv8EVZ?version=latest).

# Arquivos de vendas Externas - CSV

## Arquivo de Transações para Upload

O arquivo de transações para upload é um arquivo de texto, mais especificamente um arquivo de valores separados por ponto-e-vírgula (CSV).
As informações são guardadas em três tipos de registros, Header, Transação e Trailer.

## Registro Header

Apenas um registro deste tipo é permitido por arquivo. O registro header deve estar na primeira linha do arquivo. O registro segue o seguinte padrão:

> [Tipo de Registro];[Versão do Layout];[Período Inicial];[Período Final]

| Campo            | Tamanho | Descrição                                                                                                      |
|------------------|---------|----------------------------------------------------------------------------------------------------------------|
| Tipo de Registro | 1       | Tipo de registro. Para o Header, este valor será sempre 0.                                                     |
| Versão do layout | 3       | Identifica qual a versão do layout do arquivo. A versão mais atual é 001.                                      |
| Período Inicial  | 8       | Identifica a data que corresponde à data da transação mais antiga contemplada no arquivo. Formato “yyyyMMdd”.  |
| Período Final    | 8       | Identifica a data que corresponde à data da transação mais recente contemplada no arquivo. Formato “yyyyMMdd”. |

## Registro Transação

Cada transação feita pela loja será demonstrada nesse registro, que se repetirá quantas vezes forem necessárias, a partir da linha 2 (logo após o Header).

<aside class="warning">Atenção: o número de identificação do pedido (MerchantOrderId) não sofre alteração, se mantendo o mesmo até o final do fluxo transacional. Contudo, um número adicional (SentOrderId) pode ser gerado para o pedido e utilizado durante a transação. Esse número (SentOrderId) só será diferente em caso de adequação a regras da adquirente ou em caso de números de identificação do pedido (MerchantOrderId) repetidos.</aside>

O registro segue o seguinte padrão:

> [Tipo de Registro];[Pedido];[Afiliação];[Adquirente];[TID];[NSU/DOC];[Código de Autorização] ; [Número do Cartão];[Valor];[Quantidade de Parcelas];[Data da Venda];[Hora da venda] ; [Identificador da filial]

| Campo                    | Tipo           | Tamanho | Descrição                                                                                           |
|--------------------------|----------------|---------|-----------------------------------------------------------------------------------------------------|
| Tipo de Registro         | Domínio2       | N/A     | Tipo de registro. Para a transação, este valor será sempre 1.                                       |
| Pedido                   | Texto          | 0~64    | Número do pedido.                                                                                   |
| Afiliação1               | Inteiro3       | N/A     | Código de afiliação na adquirente.                                                                  |
| Adquirente1              | Domínio2       | N/A     | Identificador da adquirente. <br>1 = Cielo<br>2 = Redecard<br>3 = Amex<br>4 = Losango<br>5 = GetNet |
| TID                      | Texto          | 0~20    | TID da transação.                                                                                   |
| NSU/DOC1                 | Inteiro longo4 | N/A     | NSU/DOC da transação.                                                                               |
| Código de autorização    | Texto          | 0~6     | Código de autorização da transação.                                                                 |
| Número do cartão         | Texto          | 0~24    | Número do cartão utilizado na transação (mascarado conforme normas PCI).                            |
| Valor1                   | Inteiro3       | N/A     | Valor da transação em centavos.                                                                     |
| Quantidade de parcelas1  | Byte5          | N/A     | Quantidade de parcelas em que a transação foi dividida. 1 = Transações à vista                      |
| Data da venda            | Domínio2       | 8       | Data da venda da transação. Formato “yyyyMMdd”.                                                     |
| Hora da venda            | Domínio2       | 8       | Hora da venda da transação. Formato “HH:mm:ss”.                                                     |
| Identificador da filial1 | Texto          | 0~256   | Identificador da filial da loja.                                                                    |
| Documento do cliente     | Texto          | 0~32    | Numero do documento de identificação do cliente.                                                    |
| Nome do cliente          | Texto          | 0~256   | Nome do cliente.                                                                                    |
| Email do cliente         | Texto          | 0~256   | Email de contato do cliente.                                                                        |

**Legenda:**

1. Campo obrigatório.
2. O tipo de dados deste campo deve seguir restritamente o formato expresso na descrição.
3. O tipo inteiro aceita como um número válido qualquer valor entre -2.147.483.648 e 2.147.483.647. Entretanto, regras de domínio adicionais podem existir para os campos (o código de afiliação deve ser válido na adquirente, por exemplo).
4. O tipo inteiro longo aceita como um número válido qualquer valor entre – 9.223.372.036.854.775.808 e 9.223.372.036.854.775.807. Entretanto, regras de domínioadicionais podem existir para os campos (o NSU da transação deve ser válido na adquirente, por exemplo).
5. O tipo byte aceita como um número válido qualquer valor entre 0 e 255. Entretanto, regras de domínio adicionais podem existir para os campos (a parcela da transação deve ser no mínimo 1, por exemplo).

**Informações Adicionais:**

* **Campo NSU** - Quando não informado, este campo deve ser vazio.
* **Campo TID** - Quando não informado, este campo deve ser vazio.
* **Campo Hora** – Caso não tenha o valor exato, informar a hora como 00:00:00.

## Registro Trailer

Apenas um registro deste tipo é permitido por arquivo. O registro trailer deve estar na última linha do arquivo. O registro segue o seguinte padrão:

> [Tipo de Registro];[Quantidade de registros]

| Campo                   | Tamanho | Descrição                                                     |
|-------------------------|---------|---------------------------------------------------------------|
| Tipo de Registro        | 1       | Tipo de registro. Para o Trailer, este valor será sempre 2.   |
| Quantidade de registros | N/A     | Quantidade de registros (exceto Header e Trailer) no arquivo. |

## Formas de envio

Atualmente o conciliador Braspag oferece 3 (três) formas de envios para os arquivos de vendas:

* Upload Manual pela aplicação
* Transferência via SFTP
* Transferência via webservice

### Transferência Manual: Upload de arquivos na aplicação.

Pre-Requisitos:

* Acesso à internet, usuário e senha com permissão para upload habilitada.
* Acessando a URL: https://reconciliation.braspag.com.br/WebSite
* No menu superior, localize a funcionalidade: **“Área do lojista”**
* Após clicar no botão **“Área do lojista”** aparecerá um menu na lateral esquerda do backoffice:
* Localize o botão **“Upload de arquivos de Transações”**, ao clicar, aparecerá a seguinte tela:

![]({{ site.baseurl_root }}/images/braspag/conciliador/upload.png)

Agora basta selecionar a loja, e realizar o upload escolhendo o arquivo que deseja transmitir de sua máquina. Para iniciar a transmissão click em “Efetuar upload”.

### Transferência automática: Configuração de SFTP

É possivel alimentar o conciliador com os dados de vendas, através desse protocolo de rede que fornece acesso a arquivos , transferência de arquivos e gerenciamento de arquivos sobre qualquer fluxo de dados confiável.
Porém para que seja possível a realização da automação, esse processo exige alguns pré-requisitos:

* O Ambiente deverá ser configurado nos domínios do lojista.
* O lojista disponibilizará diariamente os arquivos de vendas no layout Braspag nesse domínio.
* A Braspag terá acesso liberado para se conectar e realizar as baixas

![]({{ site.baseurl_root }}/images/braspag/conciliador/fluxo.png)

### Transferência de arquivos automática via Webservice:

> URL: https://reconciliation.braspag.com.br/WebServices/ReconciliationFilesWebservice.asmx

**Método:** SendTransactionFile

| Parâmetro          | Tipo   | Descrição                        | Condição          |
|--------------------|--------|----------------------------------|-------------------|
| RequestId          | Guid   | Identificador da requisição      | Campo obrigatório |
| MerchantId         | Int    | Identificador da loja            | Campo obrigatório |
| RequestingUserName | String | Nome do usuário vinculado a loja | Campo obrigatório |
| RequestingPassword | String | Senha do usuário                 | Campo obrigatório |
| FileContent        | String | Conteúdo do arquivo em binário   | Campo obrigatório |

**OBS**: Os dados desse arquivo são codificados como um "array" de bytes, ou seja, deverá ser enviado os bytes do arquivo.

A solução para isso é utilizar a codificação Base64. Esse tipo de codificação é o padrão usado quando se deseja fazer transferência de dados binários utilizando texto como a fonte do meio de comunicação. Em outras palavras, essa codificação é capaz de representar conteúdo de texto em forma de binário.

Para gerar o conteúdo do FileContent, é preciso primeiro saber codificar um dado textual em Base64. Existem documentos na internet explicando todo o processo conversão. A maioria das linguagens de programação atualmente já possuem algum framework integrado com alguma função que faça a codificação/decodificação.

**Referência:**
Caso o cliente queira codificar/decodificar um conteúdo como teste, ele pode acessar este site:

> http://www.base64decode.org/

#### Request

![]({{ site.baseurl_root }}/images/braspag/conciliador/request.png)

#### Response

![]({{ site.baseurl_root }}/images/braspag/conciliador/response.png)

## Tipos de erro

Quando o valor da TAG `<Sucess>` for igual a false, a coleção de `<ErrorReport>` será retornada de acordo com um ou mais códigos de erros. Vide tabela abaixo.

| Código | Descrição                                                                                                |
|--------|----------------------------------------------------------------------------------------------------------|
| 24     | RequestId inválido.                                                                                      |
| 26     | Arquivo de transações de loja inválido. (Qualquer erro no conteúdo do arquivo.)                          |
| 39     | Erro interno do sistema.                                                                                 |
| 42     | Um MerchantId válido não foi especificado. É necessário especificar o identificador da loja.             |
| 43     | Um conteúdo válido não foi especificado para o aruqivo. É necessário especificar o valor de FileContent. |
| 44     | Acesso não autorizado no IP para a loja fornecida na requisição.                                         |
| 46     | O usuário é inválido ou não tem acesso à loja especificada.                                              |

### Exemplo

![]({{ site.baseurl_root }}/images/braspag/conciliador/error.png)

# Arquivos Fluxo de Caixa 2.0 - XML

## Introdução

Este manual tem como objetivo orientar o desenvolvimento do Arquivo de Fluxo de Caixa da plataforma conciliador no formato XML, e a extração do seu conteúdo através do Webservice

## Legenda para os tipos de formato

### Tipos de Formato (Tabela II)

| Descritor | Significado                                     | Exemplo                            |
|-----------|-------------------------------------------------|------------------------------------|
| N         | Um ou mais algarismos (0 a 9)                   | 243                                |
| A         | Um ou mais caracteres alfanuméricos             | Texto                              |
| {N}       | Um  único algarismo (0 a 9)                     | 2                                  |
| {A}       | Identificador Único Global (GUID)1              | B                                  |
| G         | Hora em campo data/hora (0 a 23)                | 4749e676-2507-442da1c6c25c08e2d2af |
| DT        | Data e hora, representados no formato ISO 86012 | 2015-06-09T10:08:19.1748259-03:00  |
| D         | Data, representada no formato ISO 86012         | 2015-06-09                         |

1. Um Identificador Único Global ou GUID (do inglês, Globally Unique IDentifier) é um tipo especial de identificador utilizado em aplicações de software para providenciar um número de referência padrão mundial. Como, por exemplo, em uma definição de referência interna para um tipo de ponto de acesso em uma aplicação de software ou para a criação de chaves únicas em um banco de dados. O número total de chaves únicas (2128 ou ~3.4×1038) é tão grande que a probabilidade do mesmo número se repetir é muito pequena. Considerando que o Universo Observável contém 5x1022 estrelas, cada estrela poderia ter ~6.8×1015 dos seus próprios GUIDs. Caso seu sistema não reconheça o formato GUID, poderá trata-lo como texto.

2. O formato ISO 8601 é o formato de data padrão existente na recomendação oficial para Schemas XML
(http://www.w3.org/TR/xmlschema-2/#dateTime). Ele consiste na representação da data no formato yyyy-MM-dd. Caso o campo também tenha a informação de hora, a data vem seguida da letra “T” para separa-la da hora, no formato HH:mm:ss:mmmmmmm, seguida finalmente pela diferença de fuso horário. Para mais informações da ISO 8601 podem ser consultadas em:

> http://en.wikipedia.org/wiki/ISO_8601.   

## Informações sobre as adquirentes

O principal insumo do Conciliador são os extratos eletrônicos gerados pelas adquirentes. Devido a isso, podem existir particularidades entre cada uma.   
Abaixo a estimativa de dias em que a adquirente envia os eventos no extrato eletrônico:

### Prazo de registro no extrato eletrônico (Tabela II)

| Adquirente     | Captura                                         | Pagamento       | Observação                                                                                                                                                  |
|----------------|-------------------------------------------------|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Cielo          | D+1                                             | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Rede           | D+1                                             | D -1            | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Amex           | Entre D+1 à  D+2                                | Entre D-4 à D-6 | *Desconsiderar do prazo a segunda feira, pois os extratos  não são gerados. Nestes dias o arquivo do Conciliador será disponibilizado somente com o header. |
| Getnet         | D+1                                             | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Ticket         | D+30 ajustada ao dia de recebimento do cliente  | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Sodexo         | D+30 ajustada ao dia de recebimento do cliente  | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Stone          | Cartão de Débito: D+1 - Cartão de Crédito: D+30 | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Global Collect | Cartão de Débito: D+1 - Cartão de Crédito: D+30 | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| First Data     | Cartão de Débito: D+1 - Cartão de Crédito: D+30 | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |

### Registro Header (Tabela III)

| Campo              | Nome                                   | Tipo               | Formato | Descrição                                                                                                                              |
|--------------------|----------------------------------------|--------------------|---------|----------------------------------------------------------------------------------------------------------------------------------------|
| MerchantId         | Identificador da Loja                  | Numérico (Inteiro) | N       | Contém o identificador único da loja no Conciliador                                                                                    |
| AcquirerId         | Identificador da Adquirente            | Numérico (Inteiro) | {N}     | 1 = Cielo<br>2 = Redecard<br>3 = Amex<br>5 = Getnet<br>6 = Ticket<br>7 = Stone<br>8 = Sodexo<br>9 = Global Payments<br>10 = First Data |
| GenerationDateTime | Geração do arquivo                     | Data/Hora          | DT      | Contém a data e a hora da geração do arquivo pelo Conciliador                                                                          |
| StartPeriod        | Período inicial                        | Data               | D       | Data inicial do período de conciliação contemplado pelo arquivo                                                                        |
| EndPeriod          | Período final                          | Data               | D       | Data final do período de conciliação contemplado pelo arquivo                                                                          |
| SequentialNumber   | Número sequencial                      | Numérico (Inteiro) | N       | Número sequencial que indica a ordem de processamento dos arquivos diários                                                             |
| ProcessingTypeId   | Identificador do tipo de processamento | Numérico (Inteiro) | {N}     | 1 = Arquivo diário <br> 2 = Arquivo reprocessado                                                                                       |
| ProcessingType     | Descrição do tipo de processamento     | Alfanumérico       | A       | “Daily” ou “Reprocessed” (Diário ou reprocessado)                                                                                      |
| Version            | Versão do arquivo                      | Alfanumérico       | A       | Versão do arquivo (“2.0”)                                                                                                              |

### Registro de Transação Conciliada (Tabela IV)

| Campo              | Nome                                 | Tipo               | Formato | Descrição                                      |
|--------------------|--------------------------------------|--------------------|---------|------------------------------------------------|
| ConciliationTypeId | Identificador do tipo de conciliação | Numérico (Inteiro) | {N}     | 1   = Automática <br> 2   = Manual             |
| ConciliationType   | Descrição do tipo de conciliação     | Alfanumérico       | A       | “Automatic” ou “Manual” (Automática ou manual) |

### Registro de Conciliação Manual (Tabela V)

| Campo                | Nome                              | Tipo         | Formato | Descrição                                                         |
|----------------------|-----------------------------------|--------------|---------|-------------------------------------------------------------------|
| ConciliationUserName | Usuário da conciliação manual     | Alfanumérico | A       | Login do usuário que efetuou a conciliação manual                 |
| ConciliationDateTime | Data e hora da conciliação manual | Data/Hora    | DT      | Data e hora em que a conciliação manual foi efetuada pelo usuário |

### Registro de Informação de Venda

| Campo                          | Nome                                            | Tipo                               | Formato | Descrição                                                                             |
|--------------------------------|-------------------------------------------------|------------------------------------|---------|---------------------------------------------------------------------------------------|
| TransactionId                  | Identificador Único da  Venda no Conciliador    | Identificador Único  Global (GUID) | G       | Identificador único do Conciliador para as informações de venda1                      |
| ExternalId                     | Identificador da Venda no  Sistema Transacional | Alfanumérico                       | A       | Identificador da Venda obtido a partir do Sistema Transacional no qual a Transação foi processada com a Adquirente-2                               |
| BranchId                       | Identificador da Filial                         | Alfanumérico                       | A       | Identificador da Filial da loja que processou a venda-3                                |
| AffiliationCode                | Código de Afiliação                             | Alfanumérico                       | A       | O código de afiliação da adquirente, informado nos dados da venda do cliente          |
| OrderId                        | Número do Pedido                                | Alfanumérico                       | A       | O número do pedido associado à venda no lojista-4                                      |
| AuthorizationCode              | Código de Autorização                           | Alfanumérico                       | A       | O Código de Autorização da transação que o Lojista recebeu da Adquirente              |
| SaleDate                       | Data da Venda                                   | Data                               | D       | A data em que foi realizada a venda no Lojista                                        |
| CaptureDate                    | Data da Captura                                 | Data                               | D       | A data da captura recebida pelo lojista da Adquirente                                 |
| TransactionAmount              | Valor da transação                              | Numérico (Inteiro)                 | N       | O valor da transação em centavos5                                                     |
| InstallmentCount               | Número de parcelas                              | Numérico (Inteiro)                 | N       | A quantidade de parcelas na qual a transação foi dividida                             |
| CustomerName                   | Nome do comprador                               | Alfanumérico                       | A       | O nome do comprador do produto                                                        |
| CustomerDocument               | Documento do comprador                          | Alfanumérico                       | A       | Documento de identificador do comprador (RG, CPF, etc.)                               |
| CustomerEmail                  | E-mail do comprador                             | Alfanumérico                       | A       | Endereço de e-mail do comprador                                                       |
| CardNumber                     | Número do cartão                                | Alfanumérico                       | A       | Número do cartão (crédito ou débito) utilizado na venda                               |
| Tid                            | TID                                             | Alfanumérico                       | A       | Identificador da transação ecommerce na Cielo, recebido pelo lojista                  |
| Nsu                            | NSU                                             | Número                             | N       | Número sequencial da transação na  Adquirente, recebido pelo lojista                  |
| IataAmount                     | Valor da taxa IATA                              | Número                             | N       | Valor da taxa IATA (apenas para setor aéreo), em centavos                             |
| PaymentMethodName              | Tipo de Integração                              | Alfanumérico                       | A       | Nome do meio de pagamento utilizado no caso da transação efetuada no  gateway Pagador |

1. As informações de venda são as transações enviadas pelo cliente do mundo físico, ou do  gateway/sistema transacional utilizado para efetuar as transações. São a primeira parte da conciliação. O Identificador Único da Venda pode ser utilizado para visualizar a venda no WebSite do Conciliador, preenchendo a URL:

> https://reconciliation.braspag.com.br/WebSite/Reports/TransactionDetails.aspx?SaleTransactionId=[ID]  

**OBS:**Onde o texto [ID] deve ser substituído pelo identificador informado no registro.   

2. O identificador da Venda no Sistema Transacional é o Identificador da Transação que é utilizado pelo sistema que efetuou a venda, seja ele um Gateway ou Sistema de Caixa/POS. Este valor pode ou não ser fornecido durante a importação da venda para o Conciliador. É de responsabilidade do cliente a decisão de informá-lo ou não.   
3. O identificador da Filial deve ser fornecido pelo cliente toda vez que a importação de uma venda é realizada para o Conciliador. Apesar de não haver restrições para o formato do identificador da filial (campo Alfanumérico), é obrigatório que cada Filial possua um identificador único.   
4. O Gerenciamento do Número do Pedido é de inteira responsabilidade do lojista. O Conciliador apenas armazena esta informação, mas nenhum tipo de validação é feito.   
5. O Valor da Transação não é o valor das parcelas. O valor informado aqui é o valor integral da mesma, da forma como informado pelo cliente/gateway.

### Registro de Informação da Adquirente

| Campo                          | Nome                                                               | Tipo                              | Formato | Descrição                                                                                                           |
|--------------------------------|--------------------------------------------------------------------|-----------------------------------|---------|---------------------------------------------------------------------------------------------------------------------|
| TransactionId                  | Identificador Único da Transação no Conciliador                    | Identificador Único Global (GUID) | G       | Identificador Único das Informações da Adquirente1                                                                  |
| AffiliationCode                | Código de Afiliação                                                | Numérico                          | N       | O Código de Afiliação informado na venda do Extrato                                                                 |
| SummaryNumber                  | Número do Lote                                                     | Numérico (Inteiro)                | N       | O número do Lote (Resumo de Vendas) ao qual a transação pertence na Adquirente                                      |
| CardNumber                     | Número do Cartão                                                   | Alfanumérico                      | A       | O número do cartão usado na compra2                                                                                 |
| SaleDate                       | Data da Venda                                                      | Data                              | D       | A data em que a venda foi efetuada, segundo as informações do extrato                                               |
| TransactionGrossAmount         | Valor bruto da Transação                                           | Numérico (Inteiro)                | N       | Valor bruto da transação em centavos                                                                                |
| TransactionTaxAmount           | Valor da taxa de adquirência deduzido da transação                 | Numérico (Inteiro)                | N       | Valor deduzido do valor bruto da transação como taxa de adquirência, em centavos                                    |
| TransactionNetAmount           | Valor líquido da transação                                         | Numérico (Inteiro)                | N       | Valor que será recebido pelo lojista, após a dedução da taxa de Adquirência sobre o valor bruto, em centavos        |
| RoundingInstallmentGrossAmount | Valor bruto da parcela de arredondamento                           | Numérico (Inteiro)                | N       | Valor bruto da parcela de arredondamento, em centavos3                                                              |
| RoundingInstallmentTaxAmount   | Valor da taxa de adquirência deduzido da parcela de arredondamento | Numérico (Inteiro)                | N       | Valor deduzido do valor bruto da parcela de arredondamento como taxa de adquirência, em centavos                    |
| RoundingInstallmentNetAmount   | Valor líquido da parcela de arredondamento                         | Numérico (Inteiro)                | N       | Valor que será recebido pelo lojista como valor da parcela de arredondamento, após a dedução da taxa de adquirência |
| InstallmentsGrossAmount        | Valor bruto das demais parcelas                                    | Numérico (Inteiro)                | N       | Valor bruto das demais parcelas da transação,  em centavos                                                          |
| InstallmentsTaxAmount          | Valor da taxa de adquirência deduzido das demais parcelas          | Numérico (Inteiro)                | N       | Valor deduzido do valor bruto das demais parcelas como taxa de adquirência, em centavos                             |
| InstallmentsNetAmount          | Valor líquido das demais parcelas                                  | Numérico (Inteiro)                | N       | Valor que será recebido pelo lojista como valor das demais parcelas, após a dedução da taxa de  adquirência         |
| Tax                            | Taxa de adquirência                                                | Numérico (Inteiro)                | N       | A taxa de adquirência sobre a transação4                                                                            |
| Tid                            | TID                                                                | Alfanumérico                      | A       | O identificador da transação ecommerce na adquirente Cielo                                                          |
| Nsu                            | NSU                                                                | Numérico (Inteiro)                | N       | O número sequencial da transação na Adquirente                                                                      |
| IataAmount                     | Valor da taxa IATA                                                 | Numérico (Inteiro)                | N       | O valor da taxa IATA (apenas para setor aéreo), cobrado sobre a transação, em centavos                              |
| OrderId                        | Número do Pedido                                                   | Alfanumérico                      | A       | O Número do Pedido recebido pela adquirente durante a concretização da transação                                    |
| TerminalLogicNumber            | Número lógico do Terminal                                          | Alfanumérico                      | A       | O número do terminal utilizado para efetuar a transação                                                             |
| CaptureDate                    | Data de Captura                                                    | Data                              | D       | A data da captura da transação na Adquirente                                                                        |
| SummaryIdentifierNumber        | Identificador Único do Lote                                        | Numérico (Inteiro  Longo)         | N       | O identificador único do Lote (Resumo de Vendas) na adquirente Cielo                                                |
| InstallmentCount               | Quantidade de Parcelas                                             | Númerico (Inteiro)                | N       | A quantidade de parcelas na qual a transação foi dividida na Adquirente                                             |
| AuthorizationCode              | Código de Autorização                                              | Alfanumérico                      | A       | O código de autorização da transação informado pela adquirente                                                      |
| CaptureMethodId                | Identificador do Meio de  Captura                                  | Numérico (Inteiro)                | N       | O identificador do meio tecnológico utilizado para capturar a transação na Adquirente5                              |
| CaptureMethodDescription       | Descrição do Meio de Captura                                       | Alfanumérico                      | A       | A descrição do meio tecnológico utilizado para capturar a transação na   Adquirente5                                |
| CardBrandId                    | Identificador da Bandeira                                          | Numérico (Inteiro)                | N       | O identificador da bandeira do cartão utilizado para efetuar a  transação6                                          |
| CardBrand                      | Nome da Bandeira                                                   | Alfanumérico                      | A       | O nome da bandeira do cartão utilizado para efetuar a transação6                                                    |
| CardTypeId                     | Identificador do tipo do cartão                                    | Numérico (Inteiro)                | N       | O identificador do tipo do cartão: - 1: Debit - 2: Credit                                                           |
| CardType                       | Tipo de cartão                                                     | Alfanumérico                      | A       | O nome do tipo do cartão utilizado para efetuar a transação                                                         |
| ProductIdentifierCode          | Código de Identificação do   Produto na Cielo                      | Numérico (Inteiro)                | N       | O código que identifica o   Produto da Adquirente Cielo utilizado para efetuar a transação7                         |
| ProductIdentifierDescription   | Descrição do Produto na Cielo                                      | Alfanumérico                      | A       | A descrição do Produto na Adquirente Cielo utilizado para efetuar a  transação7                                     |

1. As informações da adquirente são os dados da venda que o Conciliador recebe dos extratos eletrônicos, o meio de integração da adquirente com os sistemas externos. São a segunda parte da conciliação. O Identificador Único da Transação pode ser utilizado para visualizar a venda no WebSite do Conciliador, preenchendo a URL:   

> https://reconciliation.braspag.com.br/WebSite/Reports/TransactionDetails.aspx?AcquirerTransactionId=[I D]   

Onde o texto [ID] deve ser substituído pelo identificador informado no registro.   

2. Devido à restrições de segurança, o número do cartão será informado de forma mascarada.   
3. Quando uma transação é dividida em uma quantidade de parcelas onde o valor não pode ser distribuído igualmente entre as parcelas, surge a necessidade de colocar um valor extra em uma parcela específica. Esta parcela é chamada de parcela de arredondamento. Como exemplo, podemos adotar uma transação de R$ 100,00 dividida em três parcelas. Se a primeira parcela for a parcela de arredondamento, ela terá o valor de R$ 33,34 – enquanto as duas outras parcelas terão o valor de R$ 33,33.   
4. A taxa de adquirência normalmente é expressa em porcentagem. O campo demonstra essa porcentagem multiplicada por 100. Portanto, se o valor deste campo for expresso como 275, isto indica uma taxa de adquirência de 2,75%.   
5. Tabelas com os meios de captura informados estão disponíveis no Apêndice do Manual.   
6. Uma tabela com as bandeiras disponíveis está disponível no Apêndice do Manual.   
7. Uma tabela com os tipos de produto está disponível no Apêndice do Manual.   

### Registro de Evento

| Campo                       | Nome                                        | Tipo               | Formato | Descrição                                                                                                             |
|-----------------------------|---------------------------------------------|--------------------|---------|-----------------------------------------------------------------------------------------------------------------------|
| EventId                     | Identficador Único do Evento no Conciliador | Identificador Único Global  (GUID)     | G       | O identificador único do do  Evento no Conciliador2                                               |
| EventDate                   | Data do evento                              | Data               | D       | A data em que o evento está previsto para ser realizado, ou a data em que foi realizado (no caso de evento realizado) |
| CategoryId                  | Identificador da Categoria de  Evento       | Numérico (Inteiro) | N       | O identificador da categoria do evento3                                                                               |
| Category                    | Descrição da Categoria de  Evento           | Alfanumérico       | A       | A descrição da categoria do evento3                                                                                   |
| TypeId                      | Identificador do Tipo de Evento             | Numérico (Inteiro) | {N}     | 1 = Realizado<br>2 = Previsto<br>3 = Pendente                                                                         |
| Type                        | Descrição do Tipo de Evento                 | Alfanumérico       | A       | “Preview”, “Realized” ou “Pending” (Previsto, realizado ou pendente)                                                  |
| AffiliationCode             | Código de Afiliação                         | Numérico           | N       | O código de afiliação do estabelecimento no qual o evento foi executado                                               |
| TransactionInstallment      | Parcela da Transação                        | Numérico           | N       | A parcela da transação a qual o evento se refere4                                                                     |
| GrossAmount                 | Valor Bruto                                 | Numérico           | N       | O valor financeiro do evento contemplado, antes da dedução da taxa de adquirência, em centavos                        |
| NetAmount                   | Valor Líquido                               | Numérico           | N       | O valor líquido do evento, após a redução da taxa de adquirência, em centavos                                         |
| TaxAmount                   | Valor da Taxa                               | Numérico           | N       | O valor da taxa de adquirência sobre o evento, em centavos                                                            |
| Bank                        | Banco                                       | Numérico           | N       | O código do banco do domicílio bancário sobre o qual o evento financeiro é ou será lançado                            |
| Agency                      | Agência                                     | Numérico           | N       | O código da agência do domicílio bancário sobre o qual o evento financeiro é ou será lançado                          |
| Account                     | Número da Conta                             | Alfanumérico       | A       | O número da conta do domicílio bancário sobre o qual o evento financeiro é ou será lançado                            |
| AcquirerAdjustCode          | Código do Ajuste                            | Alfanumérico       | A       | O código que identifica o tipo de ajuste (apenas para eventos de ajustes)                                             |
| AcquirerAdjustDescription   | Descrição do Ajuste                         | Alfanumérico       | A       | A descrição do ajuste (apenas para eventos de ajustes)                                                                |
| AnticipationOperationNumber | Número da Operação de  Antecipação na Cielo | Numérico           | N       | O número da Operação de Antecipação (apenas para eventos derivados de antecipações na Cielo)                          |
| OriginalPaymentDate         | Data original de pagamento                  | Data               | D       | A data original para a qual o evento de pagamento estava previsto (apenas para eventos de antecipações)               |

## Meios de Captura

### Cielo

| Código/Identificador | Descrição           |
|----------------------|---------------------|
| 1                    | POS                 |
| 2                    | PDV/TEF             |
| 3                    | E-Commerce          |
| 4                    | EDI                 |
| 5                    | ADP/BSP             |
| 6                    | Manual              |
| 7                    | URA/CVA             |
| 8                    | Mobile              |
| 9                    | Moedeiro Eletrônico |

### Amex

| Código/Identificador | Descrição                           |
|----------------------|-------------------------------------|
| 1                    | Rede AE – Manual                    |
| 2                    | Rede AE – EDI                       |
| 3                    | Rede AE – BSP                       |
| 4                    | Rede AE – TEF                       |
| 11                   | Cielo – POS                         |
| 12                   | Cielo – TEF                         |
| 13                   | Cielo – Autorização Manual          |
| 14                   | Cielo – URA                         |
| 15                   | Cielo – EDI                         |
| 16                   | Cielo – GDS                         |
| 17                   | Cielo – E-Commerce                  |
| 18                   | Cielo – Mobile                      |
| 99                   | Legado – Versão anterior do extrato |

### Rede

| Código/Identificador | Descrição        |
|----------------------|------------------|
| 1                    | Manual           |
| 2                    | POS              |
| 3                    | PDV              |
| 4                    | TO               |
| 5                    | Internet         |
| 6                    | Leitor de Trilha |
| 9                    | Outros           |

### GetNet

| Código/Identificador | Descrição |
|----------------------|-----------|
| 0                    | TEF       |
| 1                    | POS       |
| 2                    | Manual    |
| 3                    | Internet  |

### Outros

A tabela abaixo é valida para:

* Stone
* Global Payments
* First Data
* Ticket
* Sodexo

**Observação:** Para a adquirente Losango o campo é enviado “vazio"

| Código/Identificador | Descrição  |
|----------------------|------------|
| 0                    | N/D        |
| 1                    | N/A        |
| 2                    | POS        |
| 3                    | PDV/TEF    |
| 4                    | E-Commerce |
| 5                    | EDI        |
| 6                    | Manual     |
| 7                    | Mobile     |
| 8                    | Outros     |

## Bandeiras

| Código/Identificador | Descrição               |
|----------------------|-------------------------|
| 0                    | Desconhecido/Indefinido |
| 1                    | VISA                    |
| 2                    | Mastercard              |
| 3                    | ELO                     |
| 4                    | Diners                  |
| 5                    | Cabal                   |
| 6                    | Hipercard               |
| 7                    | Amex                    |
| 8                    | Sicred                  |
| 9                    | Cup                     |
| 10                   | Agiplan                 |
| 11                   | Banesecard              |
| 12                   | SoroCred                |
| 13                   | CredSystem              |
| 14                   | Esplanada               |
| 15                   | CredZ                   |
| 16                   | Losango                 |
| 17                   | AVista                  |
| 18                   | Hiper                   |
| 19                   | JCB                     |
| 20                   | Aura                    |
| 21                   | Alelo                   |
| 22                   | Ticket                  |
| 23                   | Sodexo                  |

## Tipos de Produtos

### Cielo

| Código/Identificador | Descrição                          |
|----------------------|------------------------------------|
| 1                    | Agiplan crédito à vista            |
| 2                    | Agiplan parcelado loja             |
| 3                    | Banescard crédito à vista          |
| 4                    | Banescard parcelado loja           |
| 5                    | Esplanada crédito à vista          |
| 6                    | CredZ crédito à vista              |
| 7                    | Esplanada parcelado loja           |
| 8                    | Credz parcelado loja               |
| 9                    | Elo Crediário                      |
| 10                   | MasterCard crédito à vista         |
| 11                   | Maestro                            |
| 12                   | MasterCard parcelado loja          |
| 13                   | Elo Construcard                    |
| 14                   | Elo Agro Débito                    |
| 15                   | Elo Agro Custeio                   |
| 16                   | Elo Agro Investimento              |
| 17                   | Elo Agro Custeio + Débito          |
| 18                   | Elo Agro Investimento + Débito     |
| 19                   | Discover crédito à vista           |
| 20                   | Diners crédito à vista             |
| 21                   | Diners parcelado loja              |
| 22                   | Agro Custeio + Electron            |
| 23                   | Agro Investimento + Electron       |
| 24                   | FCO Investimento                   |
| 25                   | Agro Electron                      |
| 26                   | Agro Custeio                       |
| 27                   | Agro Investimento                  |
| 28                   | FCO Giro                           |
| 33                   | JCB                                |
| 36                   | Saque com cartão de Débito VISA    |
| 37                   | Flex Car Visa Vale                 |
| 38                   | CredSystem crédito à vista         |
| 39                   | CredSystem parcelado loja          |
| 40                   | Visa Crédito à Vista               |
| 41                   | Visa Electron Débito à Vista       |
| 42                   | Visa Pedágio                       |
| 43                   | Visa Parcelado Loja                |
| 44                   | Visa Electron Pré-Datado           |
| 45                   | Alelo Refeição (Bandeira Visa/Elo) |

### Getnet

| Código/Identificador | Descrição                        |
|----------------------|----------------------------------|
| 1                    | Título                           |
| 2                    | Convênio                         |
| 3                    | Crédito Digital                  |
| 00/CE                | Cupom Eletrônico                 |
| CP                   | Cupom Papel                      |
| SM                   | Cartão de Crédito MASTERCARD     |
| SV                   | Cartão de Crédito VISA           |
| SR                   | Cartão de Débito MAESTRO         |
| SE                   | Cartão de Débito VISA ELECTRON   |
| PV                   | Pagamento Carnê – Débito VISA    |
| ELECTRON             |                                  |
| PM                   | Pagamento Carnê – Débito MAESTRO |
| PR                   | Pagamento Recorrente             |

### Stone

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 1                    | Crédito Visa       |
| 2                    | Crédito Master     |
| 4                    | Crédito Elo        |
| 15                   | Crédito Hipercard  |
| 35                   | Mastercard Maestro |
| 36                   | Visa Electron      |
| 38                   | Débito Elo         |
| 39                   | Débito Hipercard   |

### Global Payments

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 1                    | Crédito Visa       |
| 2                    | Crédito Master     |
| 35                   | Mastercard Maestro |
| 36                   | Visa Electron      |

### First Data

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 1                    | Crédito Visa       |
| 2                    | Crédito Master     |
| 4                    | Crédito Elo        |
| 11                   | Crédito Cabal      |
| 35                   | Mastercard Maestro |
| 36                   | Visa Electron      |
| 37                   | Débito Cabal       |
| 38                   | Débito Elo         |

### Ticket

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 24                   | Ticket Refeição    |
| 25                   | Ticket Alimentação |
| 26                   | Ticket Parceiro    |
| 27                   | Ticket Cultura     |

### Sodexo

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 28                   | Sodexo Refeição    |
| 29                   | Sodexo Alimentação |
| 30                   | Sodexo Gift        |
| 31                   | Sodexo Premium     |
| 32                   | Sodexo Cultura     |

## Categorias de Evento

| Código/Identificador | Descrição                           |
|----------------------|-------------------------------------|
| 1                    | Ajuste                              |
| 2                    | POS                                 |
| 3                    | Captura                             |
| 4                    | Pagamento                           |
| 5                    | Pgto. Lote                          |
| 6                    | Aceleração                          |
| 8                    | Estorno                             |
| 9                    | Chargeback                          |
| 10                   | Antecipação                         |
| 11                   | Antecip. Lote                       |
| 13                   | Reagendamento                       |
| 14                   | Custo de Operação de Antecipação    |
| 15                   | Valor Retido                        |
| 16                   | Pagamento de Valor Retido           |
| 17                   | Débito de Valor Retido              |
| 19                   | Arredondamento de Parcelas          |
| 20                   | Estorno Antecipado                  |
| 21                   | Ajuste Indefinido                   |
| 22                   | Débito Acumulado                    |
| 25                   | Pagamento de Aceleração Antecipada  |
| 26                   | Débito de Antecipação de Aceleração |
| 29                   | Antecipação de Chargeback           |
| 30                   | Antecipação de Aluguel de POS       |
| 31                   | Antecipação de Ajustes Lote         |
| 32                   | Antecipação de Estornos Lote        |
| 33                   | Antecipação de Chargeback Lote      |

## Webservice Conciliador    

Webservice é uma solução para integrar aplicações.
Por meio dele, qualquer sistema pode se conectar para consultar ou inserir dados. Atualmente no Webservice do Conciliador é possível enviar o arquivo de vendas externas e baixar o conteúdo dos Arquivos de Fluxo de Caixa.    

Inicialmente o acesso deverá ser liberado pela equipe de operações, através do e-mail, senha e o cadastro dos IPs informados pelo estabelecimento.    

O Webservice está disponível através da URL:

> https://reconciliation.braspag.com.br/WebServices/ReconciliationFilesWebService.asmx   

**Método:** GetExportedFileV2

### Request

``` xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:rec="https://reconciliation.braspag.com.br">   
   <soapenv:Header/>   
   <soapenv:Body>   
      <rec:GetReconciliationFile>   
         <rec:request>   
            <rec:RequestId>[Guid aleatório]</rec:RequestId>   
            <rec:RequestingUserName>[Login do usuário da loja]</rec:RequestingUserName>   
            <rec:MerchantId>[Identificador da loja, fornecido pela Braspag]</rec:MerchantId>   
            <rec:AcquirerId>[Identificador da adquirente]</rec:AcquirerId>   
            <rec:RequestingPassword>[Senha do usuário da loja]</rec:RequestingPassword>   
            <rec:ReferenceDate>[Data de referência do arquivo]</rec:ReferenceDate>   
            <rec:FileExtensionType>[Extensão do arquivo]</rec:FileExtensionType>   
            <rec:FileType>[Tipo do arquivo]</rec:FileType>   
         </rec:request>   
      </rec:GetReconciliationFile>   
   </soapenv:Body>  
```

| Descritor          | Significado                                | Exemplo                                              |
|--------------------|--------------------------------------------|------------------------------------------------------|
| RequestId          | Identificador Único Global (GUID)          | 4749e676-2507-442da1c6c25c08e2d2af                   |
| RequestingUserName | Login do usuário da loja                   | user@braspag.com.br                                  |
| MerchantId         | Identificador da loja no Conciliador       | 123                                                  |
| AcquirerId         | Identificador da adquirente                | 1=Cielo<br>2=Rede<br>3=Amex<br>4=Losango<br>5=Getnet |
| RequestingPassword | Senha do usuário da loja                   | Braspag@2015                                         |
| ReferenceDate      | Data de referência do Arquivo (yyyy-mm-dd) | 12/06/2015                                           |
| FileExtensionType  | Formato do Arquivo                         | 1=CSV <br> 2=XML                                     |
| FileType           | Tipo de Arquivo                            | 1=Arquivo de Conciliação                             |

### Response

``` xml

--Bem sucedido

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchemainstance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">    <soap:Body>   
      <GetReconciliationFileResponse xmlns="https://reconciliation.braspag.com.br">   
         <GetReconciliationFileResult>   
            <CorrelatedId>124be5db-a809-47e1-b456-1f2e103caa17</CorrelatedId>   
            <Success>true</Success>               <ErrorReportCollection/>   
            <FileContent> QXJxdWl2byBkZSB0ZXN0ZSBjb25jaWxpYWRvcg==</FileContent>   
         </GetReconciliationFileResult>   
      </GetReconciliationFileResponse>   
   </soap:Body>   
</soap:Envelope>   
```

``` xml

-- Mal Sucedida   

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchemainstance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">    <soap:Body>   
      <GetReconciliationFileResponse xmlns="https://reconciliation.braspag.com.br">   
         <GetReconciliationFileResult>   
            <CorrelatedId>124be5db-a809-47e1-b456-1f2e103caa17</CorrelatedId>   
            <Success>false</Success>   
            <ErrorReportCollection>   
               <ErrorReport>   
                  <Code>44</Code>   
                  <Message>Acesso não autorizado do IP para a loja fornecida na requisição.</Message></ErrorReport>   
            </ErrorReportCollection>   
         </GetReconciliationFileResult>   
      </GetReconciliationFileResponse>   
   </soap:Body>   
</soap:Envelope>   
```

| Descritor             | Descrição                                                                                      | Exemplo                                                                                                                               |
|-----------------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| CorrelatedId          | GUID enviado na requisição                                                                     | 4749e676-2507-442d-a1c6-c25c08e2d2af                                                                                                  |
| Success               | Indica se a operação foi concluída.                                                            | false/true                                                                                                                            |
| ErrorReportCollection | Coleção de erros que será  retornada em caso de Sucess="false"                                 | ErrorReport.Code = 39,ErrorReport.Message  = “Erro interno do sistema”.                                                               |
| ErrorReport.Code      | Código de erros para  Sucess="false"                                                           | 39/44/46                                                                                                                              |
| ErrorReport.Message   | Mensagem de erro correspondente ao código informado                                            | 39 - Erro interno do sistema.<br>44 - Acesso não autorizado, IP não cadastrado<br>46 - Usuário incorreto, e/ou não tem acesso a loja. |
| FileContent           | Para requisições com Sucess="true", será enviado o conteúdo binário codificado na base64 UTF-8 | QXJxdWl2byBkZSB0ZXN0ZSBjb25jaWxpYWRvcg==                                                                                              |

## Apêndice:

### Utilizando o arquivo de Schema Definition (XSD)   

Uma forma de integração com o Arquivo de Fluxo de Caixa do Conciliador no formato XML de maneira programada ou automática, é feita através do arquivo de definição de esquema, ou `XML Schema Definition File`.   

O XML Schema é uma linguagem baseada no formato XML para definição de regras de validação ("esquemas") em documentos no formato XML. Foi a primeira linguagem de esquema para XML que obteve recomendação por parte do **W3C**. Esta linguagem é uma alternativa ao **DTD**, cuja sintaxe não é baseada no formato XML.   
Foi amplamente utilizado para desenvolvimento da NF-e (Nota Fiscal Eletrônica) Brasileira.   

Um arquivo contendo as definições na linguagem XML Schema é chamado de **XSD (XML Schema Definition)**, este descreve a estrutura de um documento XML.   

O conciliador possui arquivos XSD para cada um de seus arquivos em XML. Desta forma, é possível, programaticamente compreender a estrutura do XML, supondo que o arquivo XSD seja corretamente interpretado pela ferramenta.   

Neste manual, não serão citadas todas as formas de se trabalhar com o arquivo, entretanto, podemos demonstrar, através da utilização de uma ferramenta provida pela IDE de desenvolvimento Visual Studio, como gerar, programaticamente uma Classe em código-fonte que pode representar o conteúdo do arquivo.   

Desta forma, será possível deserializar o conteúdo de qualquer arquivo dentro desta classe, e com isto utilizar os arquivos de conciliação em um sistema com código orientado a objetos.   

Criando a classe do arquivo por meio do arquivo de definição de esquema   
Com os arquivos de definição de esquema, você deverá utilizar uma ferramenta que pode ser acessada à partir da linha de comando da IDE do Visual Studio (Visual Studio Command Prompt). O nome do executável é “xsd” (sem aspas).   

O executável possui uma série de parâmetros para customizar a geração da sua classe a partir do arquivo de definição de esquema. Caso você queira ver todas as opções, consulte a URL https://msdn.microsoft.com/en-us/library/x6c1kb0s(v=VS.100).aspx.   
No exemplo abaixo, utilizamos o comando para gerar a classe da forma mais básica. Acesse o diretório onde os arquivos de esquemas estão salvos(normalmente são dois arquivos, “ConciliationFile.xsd” e “Guid.xsd”), usando o comando CD do DOS.   Uma vez dentro deste diretório, basta executar o comando conforme abaixo:
xsd  

ConciliationFile.xsd Guid.xsd /classes   

Um exemplo do efeito disto na linha de commando é demonstrado na imagem abaixo. A classe gerada terá o nome
**“ConciliationFile_Guid.cs”**.
Esta classe pode ser inserida em um projeto na linguagem C#, e com algumas alterações pode representar o conteúdo do arquivo por meio de deserialização.   
Consulte o suporte de sua linguagem de desenvolvimento para verificar se a mesma possui algum tipo de automatização para interpretar a leitura do arquivo em XML. Isto pode facilitar seu processo de desenvolvimento e aprendizado do layout do mesmo

![]({{ site.baseurl_root }}/images/braspag/conciliador/xml20.png)

# Arquivos Fluxo de Caixa 2.0 - CSV

Este manual tem como objetivo orientar o desenvolvimento do Arquivo de Fluxo de Caixa da plataforma conciliador no formato CSV, e a extração do seu conteúdo através do Webservice.

## Legenda para os tipos de formato

| Descritor | Significado                         | Exemplo                              |
|-----------|-------------------------------------|--------------------------------------|
| N         | Um ou mais algarismos (0 a 9)       | 243                                  |
| A         | Um ou mais caracteres alfanuméricos | Texto                                |
| {N}       | Um  único algarismo (0 a 9)         | 2                                    |
| {A}       | Um único caractere alfanumérico     | B                                    |
| HH        | Hora em campo data/hora (0 a 23)    | 22                                   |
| mm        | Minuto em campo data/hora (0 a 59)  | 23                                   |
| ss        | Segundo em campo data/hora (0 a 59) | 35                                   |
| dd        | Dia em campo data/hora (1 a 31)     | 28                                   |
| MM        | Mês em campo data/hora (1 a 12)     | 10                                   |
| yyyyy     | Ano em campo data/hora              | 2015                                 |
| G         | Identificador Único Global (GUID)1  | 4749e676-2507-442d-a1c6-c25c08e2d2af |

1. Um Identificador Único Global ou GUID (do inglês, Globally Unique IDentifier) é um tipo especial de identificador utilizado em aplicações de software para providenciar um número de referência padrão mundial. Como, por exemplo, em uma definição de referência interna para um tipo de ponto de acesso em uma aplicação de software ou para a criação de chaves únicas em um banco de dados. O número total de chaves únicas (2128 ou ~3.4×1038) é tão grande que a probabilidade do mesmo número se repetir é muito pequena. Considerando que o Universo Observável contém 5x1022 estrelas, cada estrela poderia ter ~6.8×1015 dos seus próprios GUIDs. Caso seu sistema não reconheça o formato GUID, poderá trata-lo como texto

## Informações sobre as adquirentes

O principal insumo do Conciliador são os extratos eletrônicos gerados pelas adquirentes. Devido a isso, podem existir particularidades entre cada uma.      
Abaixo a estimativa de dias em que a adquirente envia os eventos no extrato eletrônico.    

### Prazo de registro no extrato eletrônico (Tabela II)

| Adquirente    | Captura                                         | Pagamento       | Observação                                                                                                                                                  |
|---------------|-------------------------------------------------|-----------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Cielo         | D+1                                             | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Rede          | D+1                                             | D -1            | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Amex          | Entre D+1 à  D+2                                | Entre D-4 à D-6 | *Desconsiderar do prazo a segunda feira, pois os extratos  não são gerados. Nestes dias o arquivo do Conciliador será disponibilizado somente com o header. |
| GetNet        | D+1                                             | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Ticket        | D+30 ajustada ao dia de recebimento do cliente  | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Sodexo        | D+30 ajustada ao dia de recebimento do cliente  | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| Stone         | Cartão de Débito: D+1 - Cartão de Crédito: D+30 | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| GlobalCollect | Cartão de Débito: D+1 - Cartão de Crédito: D+30 |                 | Pagamentos são realizados de Segunda à Sexta                                                                                                                |
| FirstData     | Cartão de Débito: D+1 - Cartão de Crédito: D+30 | D               | Pagamentos são realizados de Segunda à Sexta                                                                                                                |

### Registro Header (Tabela III)   

| Campo                                  | Tipo               | Formato        | Descrição                                                                                                                              |
|----------------------------------------|--------------------|----------------|----------------------------------------------------------------------------------------------------------------------------------------|
| Identificador do Registro              | Numérico (Inteiro) | N              | Constante com valor 1                                                                                                                  |
| Identificador da Loja                  | Numérico (Inteiro) | N              | Contém o identificador único da loja no Conciliador                                                                                    |
| Identificador da Adquirente            | Numérico (Inteiro) | {N}            | 1 = Cielo<br>2 = Redecard<br>3 = Amex<br>5 = Getnet<br>6 = Ticket<br>7 = Stone<br>8 = Sodexo<br>9 = Global Payments<br>10 = First Data |
| Geração do arquivo                     | Data/Hora          | ddMMyyyyHHmmss | Contém a data e a hora da geração do arquivo pelo Conciliador                                                                          |
| Período inicial                        | Data               | ddMMyyyy       | Data inicial do período de conciliação contemplado pelo arquivo                                                                        |
| Período final                          | Data               | ddMMyyyy       | Data final do período de conciliação contemplado pelo arquivo                                                                          |
| Número sequencial                      | Numérico (Inteiro) | N              | Número sequencial que indica a ordem de processamento dos arquivos diários                                                             |
| Identificador do tipo de processamento | Numérico (Inteiro) | {N}            | 1 = Arquivo diário  <br> 2 = Arquivo reprocessado                                                                                      |
| Descrição do tipo de processamento     | Alfanumérico       | A              | “Daily” ou “Reprocessed” (Diário ou reprocessado)                                                                                      |
| Versão do arquivo                      | Alfanumérico       | A              | Versão do arquivo (“2.0”)                                                                                                              |

### Registro de Transação Conciliada (Tabela IV)

| Campo                                | Tipo               | Formato | Descrição                          |
|--------------------------------------|--------------------|---------|------------------------------------|
| Identificador do Registro            | Numérico (Inteiro) | N       | Constante com valor  2             |
| Identificador do tipo de conciliação | Numérico (Inteiro) | {N}     | 1   = Automática  <br>2   = Manual |
| Descrição do tipo de conciliação     | Alfanumérico       | A       | “Automatic” ou  “Manual”           |

### Registro de Conciliação Manual (Tabela V)

| Campo                                | Tipo               | Formato | Descrição                          |
|--------------------------------------|--------------------|---------|------------------------------------|
| Identificador do Registro            | Numérico (Inteiro) | N       | Constante com valor  9             |
| Usuario de conciliação manual        | Alfanumérico)      | A       | Login do usuario que efetuou a conciliação manual |
| Data e hora da conciliação manual    | Data/hora          | ddmmyyyyhhmmss       | Data e hora sm que a conciliação manual foi efeturada pelo usuário  |

### Registro de Informação de Venda (Tabela VI)

| Campo                                           | Tipo                               | Formato  | Descrição                                                                                                           |
|-------------------------------------------------|------------------------------------|----------|---------------------------------------------------------------------------------------------------------------------|
| Identificador do Registro                       | Numérico (Inteiro)                 | N        | Constante com valor 3                                                                                               |
| Identificador Único da  Venda no Conciliador    | Identificador Único  Global (GUID) | G        | Identificador único do Conciliador para as informações de venda1                                                    |
| Identificador da Venda no  Sistema Transacional | Alfanumérico                       | A        | Identificador da Venda obtido a partir do Sistema transacional no qual a transação foi processada com a Adquirente² |
| Identificador da Filial                         | Alfanumérico                       | A        | Identificador da Filial da loja que processou a venda3                                                              |
| Código de Afiliação                             | Alfanumérico                       | A        | O código de afiliação da adquirente, informado nos dados da venda do cliente                                        |
| Número do Pedido                                | Alfanumérico                       | A        | O número do pedido associado à venda no lojista4                                                                    |
| Código de Autorização                           | Alfanumérico                       | A        | O Código de Autorização da transação que o Lojista recebeu da Adquirente                                            |
| Data da Venda                                   | Data                               | ddMMyyyy | A data em que foi realizada a venda no lojista                                                                      |
| Data da Captura                                 | Data                               | ddMMyyyy | A data da captura recebida pelo lojista                                                                             |
| Valor da transação                              | Numérico (Inteiro)                 | N        | O valor da transação em centavos5                                                                                   |
| Número de parcelas                              | Numérico (Inteiro)                 | N        | A quantidade de parcelas na qual a transação foi dividida                                                           |
| Nome do comprador                               | Alfanumérico                       | A        | O nome do comprador do produto                                                                                      |
| Documento do comprador                          | Alfanumérico                       | A        | Documento de identificador do comprador (RG, CPF, etc.)                                                             |
| E-mail do comprador                             | Alfanumérico                       | A        | Endereço de e-mail do comprador                                                                                     |
| Número do cartão                                | Alfanumérico                       | A        | Número do cartão (crédito ou débito) utilizado na venda                                                             |
| TID                                             | Alfanumérico                       | A        | Identificador da transação e-commerce na Cielo, recebido pelo lojista                                               |
| NSU                                             | Número                             | N        | Número sequencial da transação na  Adquirente, recebido pelo lojista                                                |
| Valor da taxa IATA                              | Número                             | N        | Valor da taxa IATA (apenas para setor aéreo), em centavos                                                           |
| Tipo de Integração                              | Alfanumérico                       | A        | Nome do meio de pagamento utilizado no caso da transação efetuada no  gateway Pagador                               |

1. As informações de venda são as transações enviadas pelo cliente do mundo físico, ou do  gateway/sistema transacional utilizado para efetuar as transações. São a primeira parte da conciliação. O Identificador Único da Venda pode ser utilizado para visualizar a venda no WebSite do Conciliador, preenchendo a URL:   

> https://reconciliation.braspag.com.br/WebSite/Reports/TransactionDetails.aspx?SaleTransactionId=[ID]

Onde o texto [ID] deve ser substituído pelo identificador informado no registro.   

2. O identificador da Venda no Sistema Transacional é o Identificador da Transação que é utilizado pelo sistema que efetuou a venda, seja ele um Gateway ou Sistema de Caixa/POS. Este valor pode ou não ser fornecido durante a importação da venda para o Conciliador. É de responsabilidade do cliente a decisão de informá-lo ou não.   
3. O identificador da Filial deve ser fornecido pelo cliente toda vez que a importação de uma venda é realizada para o Conciliador. Apesar de não haver restrições para o formato do identificador da filial (campo Alfanumérico), é obrigatório que cada Filial possua um identificador único.   
4. O Gerenciamento do Número do Pedido é de inteira responsabilidade do lojista. O Conciliador apenas armazena esta informação, mas nenhum tipo de validação é feito.   
5. O Valor da Transação não é o valor das parcelas. O valor informado aqui é o valor integral da mesma, da forma como informado pelo cliente/gateway.   

### Registro de Informação da Adquirente (Tabela VII)

| Campo                                                              | Tipo                               | Formato  | Descrição                                                                                                            |
|--------------------------------------------------------------------|------------------------------------|----------|----------------------------------------------------------------------------------------------------------------------|
| Identificador do Registro                                          | Numérico (Inteiro)                 | N        | Constante com valor 4                                                                                                |
| Identificador Único da  Transação no Conciliador                   | Identificador Único  Global (GUID) | G        | Identificador Único das  informações da Adquirente                                                                   |
| Código de Afiliação                                                | Numérico                           | N        | O Código de Afiliação informado na venda do Extrato.                                                                 |
| Número do Lote                                                     | Numérico (Inteiro)                 | N        | O número do Lote (Resumo de Vendas) ao qual a transação pertence na Adquirente.                                      |
| Número do Cartão                                                   | Alfanumérico                       | A        | O número do cartão usado na compra2                                                                                  |
| Data da Venda                                                      | Data                               | ddMMyyyy | A data em que a venda foi efetuada, segundo as informações do extrato.                                               |
| Valor bruto da Transação                                           | Numérico (Inteiro)                 | N        | Valor bruto da transação em centavos.                                                                                |
| Valor da taxa de adquirência deduzido da transação                 | Numérico (Inteiro)                 | N        | Valor deduzido do valor bruto da transação como taxa de adquirência, em centavos.                                    |
| Valor líquido da transação                                         | Numérico (Inteiro)                 | N        | Valor que será recebido pelo lojista, após a dedução da taxa de Adquirência sobre o valor bruto, em centavos.        |
| Valor bruto da parcela de arredondamento                           | Numérico (Inteiro)                 | N        | Valor bruto da parcela de arredondamento, em centavos3                                                               |
| Valor da taxa de adquirência deduzido da parcela de arredondamento | Numérico (Inteiro)                 | N        | Valor deduzido do valor bruto da parcela de arredondamento como taxa de adquirência, em centavos.                    |
| Valor líquido da parcela de arredondamento                         | Numérico (Inteiro)                 | N        | Valor que será recebido pelo lojista como valor da parcela de arredondamento, após a dedução da taxa de adquirência. |
| Valor bruto das demais parcelas                                    | Numérico (Inteiro)                 | N        | Valor bruto das demais parcelas da transação, em centavos.                                                           |
| Valor da taxa de adquirência deduzido das demais parcelas          | Numérico (Inteiro)                 | N        | Valor deduzido do valor bruto das demais parcelas como taxa de adquirência, em centavos.                             |
| Valor líquido das demais parcelas                                  | Numérico (Inteiro)                 | N        | Valor que será recebido pelo lojista como valor das demais parcelas, após a dedução da taxa de adquirência.          |
| Taxa de adquirência                                                | Numérico (Inteiro)                 | N        | A taxa de adquirência sobre a transação                                                                              |
| TID                                                                | Alfanumérico                       | A        | O identificador da transação ecommerce na adquirente Cielo.                                                          |
| NSU                                                                | Numérico (Inteiro)                 | N        | O número sequencial da transação na Adquirente                                                                       |
| Valor da taxa IATA                                                 | Numérico (Inteiro)                 | N        | O valor da taxa IATA (apenas para setor aéreo), cobrado sobre a transação, em centavos.                              |
| Número do Pedido                                                   | Alfanumérico                       | A        | O Número do Pedido recebido pela adquirente durante a concretização da transação.                                    |
| Número lógico do Terminal                                          | Alfanumérico                       | A        | O número do terminal utilizado para efetuar a transação.                                                             |
| Data de Captura                                                    | Data                               | ddMMyyyy | A data da captura da transação na Adquirente.                                                                        |
| Identificador Único do Lote                                        | Numérico (Inteiro  Longo)          | N        | O identificador único do Lote (Resumo de Vendas) na adquirente Cielo                                                 |
| Quantidade de Parcelas                                             | Númerico (Inteiro)                 | N        | A quantidade de parcelas na qual a transação foi dividida na Adquirente.                                             |
| Código de Autorização                                              | Alfanumérico                       | A        | O código de autorização da transação informado pela adquirente.                                                      |
| Identificador do Meio de  Captura                                  | Numérico (Inteiro)                 | N        | O identificador do meio  tecnológico utilizado para capturar a transação na   Adquirente5                            |
| Descrição do Meio de Captura                                       | Alfanumérico                       | A        | A descrição do meio tecnológico utilizado para capturar a transação na  Adquirente5                                  |
| Identificador da Bandeira                                          | Numérico (Inteiro)                 | N        | O identificador da bandeira do cartão utilizado para efetuar a transação6                                            |
| Nome da Bandeira                                                   | Alfanumérico                       | A        | O nome da bandeira do cartão utilizado para efetuar a transação6                                                     |
| Identificador do tipo de cartão                                    | Numérico (Inteiro)                 | N        | O identificador do tipo do cartão: - 1: Debit  - 2: Credit                                                           |
| Nome do tipo de cartão                                             | Tipo de cartão                     | A        | O nome do tipo do cartão utilizado para efetuar a transação                                                          |
| Código de Identificação do  Produto na Cielo                       | Numérico (Inteiro)                 | N        | O código que identifica o   Produto da Adquirente Cielo utilizado para efetuar a transação7                          |
| Descrição do Produto na Cielo                                      | Alfanumérico                       | A        | A descrição do Produto na Adquirente Cielo utilizado para efetuar a transação7                                       |

1. As informações da adquirente são os dados da venda que o Conciliador recebe dos extratos eletrônicos, o meio de integração da adquirente com os sistemas externos. São a segunda parte da conciliação. O Identificador Único da Transação pode ser utilizado para visualizar a venda no WebSite do Conciliador, preenchendo a URL:   

> https://reconciliation.braspag.com.br/WebSite/Reports/TransactionDetails.aspx?AcquirerTransactionId=[I D]   

Onde o texto [ID] deve ser substituído pelo identificador informado no registro.   

2. Devido à restrições de segurança, o número do cartão será informado de forma mascarada.   
3. Quando uma transação é dividida em uma quantidade de parcelas onde o valor não pode ser distribuído igualmente entre as parcelas, surge a necessidade de colocar um valor extra em uma parcela específica. Esta parcela é chamada de parcela de arredondamento. Como exemplo, podemos adotar uma transação de R$ 100,00 dividida em três parcelas. Se a primeira parcela for a parcela de arredondamento, ela terá o valor de R$ 33,34 – enquanto as duas outras parcelas terão o valor de R$ 33,33.   
4. A taxa de adquirência normalmente é expressa em porcentagem. O campo demonstra essa porcentagem multiplicada por 100. Portanto, se o valor deste campo for expresso como 275, isto indica uma taxa de adquirência de 2,75%.   
5. Tabelas com os meios de captura informados estão disponíveis no Apêndice do Manual.   
6. Uma tabela com as bandeiras disponíveis está disponível no Apêndice do Manual.   
7. Uma tabela com os tipos de produto está disponível no Apêndice do Manual.  

### Registro de Evento

| Campo                                         | Tipo                               | Formato  | Descrição                                                                                                                                                          |
|-----------------------------------------------|------------------------------------|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Identificador do Registro                     | Numérico (Inteiro)                 | N        | 5 = Evento Financeiro de Transação <br> 6 = Evento Informativo de Transação <br> 7 = Eventos financeiros de Afiliação <br> 8 = Eventos  informativos de Afiliação1 |
| Identificador Único do Evento no  Conciliador | Identificador Único  Global (GUID) | G        | O identificador único do do  Evento no Conciliador2                                                                                                                |
| Data do evento                                | Data                               | ddMMyyyy | A data em que o evento está previsto para ser realizado, ou a data em que foi realizado (no caso de evento realizado)                                              |
| Identificador da Categoria de Evento          | Numérico (Inteiro)                 | N        | O identificador da categoria do evento3                                                                                                                            |
| Descrição da Categoria de Evento              | Alfanumérico                       | A        | A descrição da categoria do evento3                                                                                                                                |
| Identificador do Tipo de Evento               | Numérico (Inteiro)                 | {N}      | 1   = Realizado <br> 2 = Previsto <br> 3 = Pendente                                                                                                                |
| Descrição do Tipo de Evento                   | Alfanumérico                       | A        | “Preview”, “Realized” ou “Pending”                                                                                                                                 |
| Código de Afiliação                           | Numérico                           | N        | O código de afiliação do estabelecimento no qual o evento foi executado                                                                                            |
| Parcela da Transação                          | Numérico                           | N        | A parcela da transação a qual o evento se refere4                                                                                                                  |
| Valor Bruto                                   | Numérico                           | N        | O valor financeiro do evento contemplado, antes da dedução da taxa de adquirência, em centavos.                                                                    |
| Valor Líquido                                 | Numérico                           | N        | O valor líquido do evento, após a redução da taxa de adquirência, em centavos.                                                                                     |
| Valor da Taxa                                 | Numérico                           | N        | O valor da taxa de adquirência sobre o evento, em centavos                                                                                                         |
| Banco                                         | Numérico                           | N        | O código do banco do domicílio bancário sobre o qual o evento financeiro é ou será lançado                                                                         |
| Agência                                       | Numérico                           | N        | O código da agência do domicílio bancário sobre o qual o evento financeiro é ou será lançado                                                                       |
| Número da Conta                               | Alfanumérico                       | A        | O número da conta do domicílio bancário sobre o qual o evento financeiro é ou será lançado                                                                         |
| Código do Ajuste                              | Alfanumérico                       | A        | O código que identifica o tipo de ajuste (apenas para eventos de ajustes)                                                                                          |
| Descrição do Ajuste                           | Alfanumérico                       | A        | A descrição do ajuste (apenas para eventos de ajustes)                                                                                                             |
| Número da Operação de Antecipação na Cielo    | Numérico                           | N        | O número da Operação de Antecipação (apenas para eventos derivados de antecipações na Cielo)                                                                       |
| Data original de pagamento                    | Data                               | ddMMyyyy | A data original para a qual o evento de pagamento estava previsto (apenas para eventos de antecipações)                                                            |
|Número do Lote                                 | Numérico                           | N        | O número do Lote (Resumo de Vendas) ao qual o evento pertence (Somente disponível no CSV a partir do dia 16/04/2018)

1. Os eventos são divididos em 2 tipos: Eventos financeiros são eventos que informam cobranças ou pagamentos na agenda financeira da adquirente com o lojista. Eventos informacionais são avisos de alterações na agenda (reagendamentos de eventos financeiros) ou outras informações não financeiras (captura de transação). Além disso, eles podem estar ligados à transações ou não. Caso estejam ligados à transações, eles aparecerão logo abaixo dos registros de transações aos quais se referem. Caso contrário, serão mencionados no final do arquivo. Utilize corretamente o identificador do registro para saber como tratar estes eventos pela sua ligação.   
2. O Identificador Único do Evento pode ser utilizado para visualizar os dados do evento no WebSite do Conciliador, preenchendo a URL:  https://reconciliation.braspag.com.br/WebSite/Reports/EventDetails.aspx?Id=[ID] Onde o texto [ID] deve ser substituído pelo identificador informado no registro.   
3. Uma tabela com as categorias de evento está disponível no apêndice do Manual.   
4. Certos eventos podem não se referir à uma parcela da transação, como Estornos ou Chargebacks.   
Nesses casos, o campo Parcela da Transação ficará vazio.   

## Meios de Captura

### Cielo

| Código/Identificador | Descrição           |
|----------------------|---------------------|
| 1                    | POS                 |
| 2                    | PDV/TEF             |
| 3                    | E-Commerce          |
| 4                    | EDI                 |
| 5                    | ADP/BSP             |
| 6                    | Manual              |
| 7                    | URA/CVA             |
| 8                    | Mobile              |
| 9                    | Moedeiro Eletrônico |

### Amex

| Código/Identificador | Descrição                           |
|----------------------|-------------------------------------|
| 1                    | Rede AE – Manual                    |
| 2                    | Rede AE – EDI                       |
| 3                    | Rede AE – BSP                       |
| 4                    | Rede AE – TEF                       |
| 11                   | Cielo – POS                         |
| 12                   | Cielo – TEF                         |
| 13                   | Cielo – Autorização Manual          |
| 14                   | Cielo – URA                         |
| 15                   | Cielo – EDI                         |
| 16                   | Cielo – GDS                         |
| 17                   | Cielo – E-Commerce                  |
| 18                   | Cielo – Mobile                      |
| 99                   | Legado – Versão anterior do extrato |

### Rede

| Código/Identificador | Descrição        |
|----------------------|------------------|
| 1                    | Manual           |
| 2                    | POS              |
| 3                    | PDV              |
| 4                    | TO               |
| 5                    | Internet         |
| 6                    | Leitor de Trilha |
| 9                    | Outros           |

### GetNet

| Código/Identificador | Descrição |
|----------------------|-----------|
| 0                    | TEF       |
| 1                    | POS       |
| 2                    | Manual    |
| 3                    | Internet  |

### Outros

A tabela abaixo é valida para:

* Stone
* Global Payments
* First Data
* Ticket
* Sodexo

**Observação:** Para a adquirente Losango o campo é enviado “vazio"

| Código/Identificador | Descrição  |
|----------------------|------------|
| 0                    | N/D        |
| 1                    | N/A        |
| 2                    | POS        |
| 3                    | PDV/TEF    |
| 4                    | E-Commerce |
| 5                    | EDI        |
| 6                    | Manual     |
| 7                    | Mobile     |
| 8                    | Outros     |

## Bandeiras

| Código/Identificador | Descrição               |
|----------------------|-------------------------|
| 0                    | Desconhecido/Indefinido |
| 1                    | VISA                    |
| 2                    | Mastercard              |
| 3                    | ELO                     |
| 4                    | Diners                  |
| 5                    | Cabal                   |
| 6                    | Hipercard               |
| 7                    | Amex                    |
| 8                    | Sicred                  |
| 9                    | Cup                     |
| 10                   | Agiplan                 |
| 11                   | Banesecard              |
| 12                   | SoroCred                |
| 13                   | CredSystem              |
| 14                   | Esplanada               |
| 15                   | CredZ                   |
| 16                   | Losango                 |
| 17                   | AVista                  |
| 18                   | Hiper                   |
| 19                   | JCB                     |
| 20                   | Aura                    |
| 21                   | Alelo                   |
| 22                   | Ticket                  |
| 23                   | Sodexo                  |

## Tipos de Produtos

### Cielo

| Código/Identificador | Descrição                          |
|----------------------|------------------------------------|
| 1                    | Agiplan crédito à vista            |
| 2                    | Agiplan parcelado loja             |
| 3                    | Banescard crédito à vista          |
| 4                    | Banescard parcelado loja           |
| 5                    | Esplanada crédito à vista          |
| 6                    | CredZ crédito à vista              |
| 7                    | Esplanada parcelado loja           |
| 8                    | Credz parcelado loja               |
| 9                    | Elo Crediário                      |
| 10                   | MasterCard crédito à vista         |
| 11                   | Maestro                            |
| 12                   | MasterCard parcelado loja          |
| 13                   | Elo Construcard                    |
| 14                   | Elo Agro Débito                    |
| 15                   | Elo Agro Custeio                   |
| 16                   | Elo Agro Investimento              |
| 17                   | Elo Agro Custeio + Débito          |
| 18                   | Elo Agro Investimento + Débito     |
| 19                   | Discover crédito à vista           |
| 20                   | Diners crédito à vista             |
| 21                   | Diners parcelado loja              |
| 22                   | Agro Custeio + Electron            |
| 23                   | Agro Investimento + Electron       |
| 24                   | FCO Investimento                   |
| 25                   | Agro Electron                      |
| 26                   | Agro Custeio                       |
| 27                   | Agro Investimento                  |
| 28                   | FCO Giro                           |
| 33                   | JCB                                |
| 36                   | Saque com cartão de Débito VISA    |
| 37                   | Flex Car Visa Vale                 |
| 38                   | CredSystem crédito à vista         |
| 39                   | CredSystem parcelado loja          |
| 40                   | Visa Crédito à Vista               |
| 41                   | Visa Electron Débito à Vista       |
| 42                   | Visa Pedágio                       |
| 43                   | Visa Parcelado Loja                |
| 44                   | Visa Electron Pré-Datado           |
| 45                   | Alelo Refeição (Bandeira Visa/Elo) |

### Getnet

| Código/Identificador | Descrição                        |
|----------------------|----------------------------------|
| 1                    | Título                           |
| 2                    | Convênio                         |
| 3                    | Crédito Digital                  |
| 00/CE                | Cupom Eletrônico                 |
| CP                   | Cupom Papel                      |
| SM                   | Cartão de Crédito MASTERCARD     |
| SV                   | Cartão de Crédito VISA           |
| SR                   | Cartão de Débito MAESTRO         |
| SE                   | Cartão de Débito VISA ELECTRON   |
| PV                   | Pagamento Carnê – Débito VISA    |
| ELECTRON             |                                  |
| PM                   | Pagamento Carnê – Débito MAESTRO |
| PR                   | Pagamento Recorrente             |

### Stone

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 1                    | Crédito Visa       |
| 2                    | Crédito Master     |
| 4                    | Crédito Elo        |
| 15                   | Crédito Hipercard  |
| 35                   | Mastercard Maestro |
| 36                   | Visa Electron      |
| 38                   | Débito Elo         |
| 39                   | Débito Hipercard   |

### Global Payments

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 1                    | Crédito Visa       |
| 2                    | Crédito Master     |
| 35                   | Mastercard Maestro |
| 36                   | Visa Electron      |

### First Data

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 1                    | Crédito Visa       |
| 2                    | Crédito Master     |
| 4                    | Crédito Elo        |
| 11                   | Crédito Cabal      |
| 35                   | Mastercard Maestro |
| 36                   | Visa Electron      |
| 37                   | Débito Cabal       |
| 38                   | Débito Elo         |

### Ticket

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 24                   | Ticket Refeição    |
| 25                   | Ticket Alimentação |
| 26                   | Ticket Parceiro    |
| 27                   | Ticket Cultura     |

### Sodexo

| Código/Identificador | Descrição          |
|----------------------|--------------------|
| 28                   | Sodexo Refeição    |
| 29                   | Sodexo Alimentação |
| 30                   | Sodexo Gift        |
| 31                   | Sodexo Premium     |
| 32                   | Sodexo Cultura     |

## Categorias de Evento

| Código/Identificador | Descrição                           |
|----------------------|-------------------------------------|
| 1                    | Ajuste                              |
| 2                    | POS                                 |
| 3                    | Captura                             |
| 4                    | Pagamento                           |
| 5                    | Pgto. Lote                          |
| 6                    | Aceleração                          |
| 8                    | Estorno                             |
| 9                    | Chargeback                          |
| 10                   | Antecipação                         |
| 11                   | Antecip. Lote                       |
| 13                   | Reagendamento                       |
| 14                   | Custo de Operação de Antecipação    |
| 15                   | Valor Retido                        |
| 16                   | Pagamento de Valor Retido           |
| 17                   | Débito de Valor Retido              |
| 19                   | Arredondamento de Parcelas          |
| 20                   | Estorno Antecipado                  |
| 21                   | Ajuste Indefinido                   |
| 22                   | Débito Acumulado                    |
| 25                   | Pagamento de Aceleração Antecipada  |
| 26                   | Débito de Antecipação de Aceleração |
| 29                   | Antecipação de Chargeback           |
| 30                   | Antecipação de Aluguel de POS       |
| 31                   | Antecipação de Ajustes Lote         |
| 32                   | Antecipação de Estornos Lote        |
| 33                   | Antecipação de Chargeback Lote      |

## Webservice Conciliador    

Webservice é uma solução para integrar aplicações.
Por meio dele, qualquer sistema pode se conectar para consultar ou inserir dados. Atualmente no Webservice do Conciliador é possível enviar o arquivo de vendas externas e baixar o conteúdo dos Arquivos de Fluxo de Caixa.    

Inicialmente o acesso deverá ser liberado pela equipe de operações, através do e-mail, senha e o cadastro dos IPs informados pelo estabelecimento.    

O Webservice está disponível através da URL:

> https://reconciliation.braspag.com.br/WebServices/ReconciliationFilesWebService.asmx   

**Método:** GetExportedFileV2

### Request

``` xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:rec="https://reconciliation.braspag.com.br">   
   <soapenv:Header/>   
   <soapenv:Body>   
      <rec:GetReconciliationFile>   
         <rec:request>   
            <rec:RequestId>[Guid aleatório]</rec:RequestId>   
            <rec:RequestingUserName>[Login do usuário da loja]</rec:RequestingUserName>   
            <rec:MerchantId>[Identificador da loja, fornecido pela Braspag]</rec:MerchantId>   
            <rec:AcquirerId>[Identificador da adquirente]</rec:AcquirerId>   
            <rec:RequestingPassword>[Senha do usuário da loja]</rec:RequestingPassword>   
            <rec:ReferenceDate>[Data de referência do arquivo]</rec:ReferenceDate>   
            <rec:FileExtensionType>[Extensão do arquivo]</rec:FileExtensionType>   
            <rec:FileType>[Tipo do arquivo]</rec:FileType>   
         </rec:request>   
      </rec:GetReconciliationFile>   
   </soapenv:Body>  
```

| Descritor          | Significado                                | Exemplo                                              |
|--------------------|--------------------------------------------|------------------------------------------------------|
| RequestId          | Identificador Único Global (GUID)          | 4749e676-2507-442da1c6c25c08e2d2af                   |
| RequestingUserName | Login do usuário da loja                   | user@braspag.com.br                                  |
| MerchantId         | Identificador da loja no Conciliador       | 123                                                  |
| AcquirerId         | Identificador da adquirente                | 1=Cielo<br>2=Rede<br>3=Amex<br>4=Losango<br>5=Getnet |
| RequestingPassword | Senha do usuário da loja                   | Braspag@2015                                         |
| ReferenceDate      | Data de referência do Arquivo (yyyy-mm-dd) | 12/06/2015                                           |
| FileExtensionType  | Formato do Arquivo                         | 1=CSV <br> 2=XML                                     |
| FileType           | Tipo de Arquivo                            | 1=Arquivo de Conciliação                             |

### Response

``` xml

--Bem sucedido

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchemainstance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">    <soap:Body>   
      <GetReconciliationFileResponse xmlns="https://reconciliation.braspag.com.br">   
         <GetReconciliationFileResult>   
            <CorrelatedId>124be5db-a809-47e1-b456-1f2e103caa17</CorrelatedId>   
            <Success>true</Success>               <ErrorReportCollection/>   
            <FileContent> QXJxdWl2byBkZSB0ZXN0ZSBjb25jaWxpYWRvcg==</FileContent>   
         </GetReconciliationFileResult>   
      </GetReconciliationFileResponse>   
   </soap:Body>   
</soap:Envelope>   
```

``` xml

-- Mal Sucedida   

<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" xmlns:xsi="http://www.w3.org/2001/XMLSchemainstance" xmlns:xsd="http://www.w3.org/2001/XMLSchema">    <soap:Body>   
      <GetReconciliationFileResponse xmlns="https://reconciliation.braspag.com.br">   
         <GetReconciliationFileResult>   
            <CorrelatedId>124be5db-a809-47e1-b456-1f2e103caa17</CorrelatedId>   
            <Success>false</Success>   
            <ErrorReportCollection>   
               <ErrorReport>   
                  <Code>44</Code>   
                  <Message>Acesso não autorizado do IP para a loja fornecida na requisição.</Message></ErrorReport>   
            </ErrorReportCollection>   
         </GetReconciliationFileResult>   
      </GetReconciliationFileResponse>   
   </soap:Body>   
</soap:Envelope>   
```

| Descritor             | Descrição                                                                                      | Exemplo                                                                                                                               |
|-----------------------|------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------|
| CorrelatedId          | GUID enviado na requisição                                                                     | 4749e676-2507-442d-a1c6-c25c08e2d2af                                                                                                  |
| Success               | Indica se a operação foi concluída.                                                            | false/true                                                                                                                            |
| ErrorReportCollection | Coleção de erros que será  retornada em caso de Sucess="false"                                 | ErrorReport.Code = 39,ErrorReport.Message  = “Erro interno do sistema”.                                                               |
| ErrorReport.Code      | Código de erros para  Sucess="false"                                                           | 39/44/46                                                                                                                              |
| ErrorReport.Message   | Mensagem de erro correspondente ao código informado                                            | 39 - Erro interno do sistema.<br>44 - Acesso não autorizado, IP não cadastrado<br>46 - Usuário incorreto, e/ou não tem acesso a loja. |
| FileContent           | Para requisições com Sucess="true", será enviado o conteúdo binário codificado na base64 UTF-8 | QXJxdWl2byBkZSB0ZXN0ZSBjb25jaWxpYWRvcg== | |

## Apêndice:

### Utilizando o arquivo de Schema Definition (XSD)   

Uma forma de integração com o Arquivo de Fluxo de Caixa do Conciliador no formato XML de maneira programada ou automática, é feita através do arquivo de definição de esquema, ou `XML Schema Definition File`.   

O XML Schema é uma linguagem baseada no formato XML para definição de regras de validação ("esquemas") em documentos no formato XML. Foi a primeira linguagem de esquema para XML que obteve recomendação por parte do **W3C**. Esta linguagem é uma alternativa ao **DTD**, cuja sintaxe não é baseada no formato XML.   
Foi amplamente utilizado para desenvolvimento da NF-e (Nota Fiscal Eletrônica) Brasileira.   

Um arquivo contendo as definições na linguagem XML Schema é chamado de **XSD (XML Schema Definition)**, este descreve a estrutura de um documento XML.   

O conciliador possui arquivos XSD para cada um de seus arquivos em XML. Desta forma, é possível, programaticamente compreender a estrutura do XML, supondo que o arquivo XSD seja corretamente interpretado pela ferramenta.   

Neste manual, não serão citadas todas as formas de se trabalhar com o arquivo, entretanto, podemos demonstrar, através da utilização de uma ferramenta provida pela IDE de desenvolvimento Visual Studio, como gerar, programaticamente uma Classe em código-fonte que pode representar o conteúdo do arquivo.   

Desta forma, será possível deserializar o conteúdo de qualquer arquivo dentro desta classe, e com isto utilizar os arquivos de conciliação em um sistema com código orientado a objetos.   

Criando a classe do arquivo por meio do arquivo de definição de esquema   
Com os arquivos de definição de esquema, você deverá utilizar uma ferramenta que pode ser acessada à partir da linha de comando da IDE do Visual Studio (Visual Studio Command Prompt). O nome do executável é “xsd” (sem aspas).   

O executável possui uma série de parâmetros para customizar a geração da sua classe a partir do arquivo de definição de esquema. Caso você queira ver todas as opções, consulte a URL https://msdn.microsoft.com/en-us/library/x6c1kb0s(v=VS.100).aspx.   
No exemplo abaixo, utilizamos o comando para gerar a classe da forma mais básica. Acesse o diretório onde os arquivos de esquemas estão salvos(normalmente são dois arquivos, “ConciliationFile.xsd” e “Guid.xsd”), usando o comando CD do DOS.   Uma vez dentro deste diretório, basta executar o comando conforme abaixo:
xsd  

ConciliationFile.xsd Guid.xsd /classes   

Um exemplo do efeito disto na linha de commando é demonstrado na imagem abaixo. A classe gerada terá o nome
**“ConciliationFile_Guid.cs”**.
Esta classe pode ser inserida em um projeto na linguagem C#, e com algumas alterações pode representar o conteúdo do arquivo por meio de deserialização.   
Consulte o suporte de sua linguagem de desenvolvimento para verificar se a mesma possui algum tipo de automatização para interpretar a leitura do arquivo em XML. Isto pode facilitar seu processo de desenvolvimento e aprendizado do layout do mesmo

![]({{ site.baseurl_root }}/images/braspag/conciliador/xml20.png)
