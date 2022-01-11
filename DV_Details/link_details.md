# Detalhamento de Links

## Links representam os relacionamentos entres chaves unicas de negócio no modelo DV2.0, e não incluem dados descritivos. Seu formato é bem parecido com o de uma `Factless-Fact` (Um dos tipos de fato propostas por Ralph Kimball).

### Relacionamentos e associações:
- Link padrão ou Standard Link (Foreign Key e Primary Key)

| LINK              | 
|-------------------|
| LINK_HASH_KEY PK  | 
| HK_HUB1       FK  | 
| HK_HUB2       FK  | 
| LOAD_DTS          | 
| REC_SRC           |

> Caracteristicas: relacionamentos unicos, não incluem dados descritivos, relacionamentos com o passado, presente e futuro, Links não contém campos de `start_dt` e `end_dt` como em dimensões SCD II. E os principais campos contidos em uma link table são colunas chaves com hash originadas em tabelas relavantes para o modelo, a origem do registro e um campo de data que guarda a primeira vez esse relacionamento foi feito no DV, e finalmente a `Hash Key Column`, que é um identificador unico da tabela gerado por um calculo baseado nas BK originadas em outras tabelas.


### Hierarquias e redefinições
- Relacionamentos hierarquicos são modelados por HAL (Hierarchical Link)
- Redefinições são modelados por SAL (same-as Link)

### Transações e eventos
- Links não historicos (possuem diferentes modelagens para Hubs e Links)
- Pedidos de venda ou dados de sensor

# Veja o exemplo abaixo

![](../src/img/3NF_Sales.png)

_Imagine o cenário acima como uma loja de eletronicos vendendo um celular ou um tablet, e para realizar esta venda e armazenar informações relevantes sobre este evento, sistematicamente nós tempos um conjunto de tabelas que usam a 3ª forma normal em um banco de dados._

### __Observe que este modelo foi modelado de forma tradicional utilizando `3NF`. Vamos detalhar as informações presentes neste modelo:__

1. Olhando para tabela de vendas, podemos ver que ela cumpre a função de ter um relacionamento muitos para muitos (N..N) entre a tabela de clientes e a tabela de produtos, agindo como uma tabela ponte e também inclui dados da transação.

2. Olhando para o relacionamento entre a tabela de clientes e a tabela de tipos de clientes, podemos ver que existe uma relação hierarquica no sentido da tabela de tipos ser uma tabela `PAI` e a tabela de clientes é por sua vez a tabela `FILHA`, que significa que um unico tipo de cliente podem haver multiplos clientes. 

No caso 2, claramente quando pensamos na modelagem DV, nós precisamos de uma `Standard Link` entre estas duas entidades:

![](../src/img/hub_customer_link.png)

_____

# Detalhamento de transações
### __Agora que analisamos parcialmente um modelo e entendemos o papel de alguns relacionamentos e como são tratados ao pensar na modelagem DV, vamos à alguns detalhes:__

__Estudo de caso:__

Existe uma sorveteria em uma rua qualquer e nós queremos ir até lá comprar sorvete de ninho trufado. 
> Esta simples ação cria entidades em um banco de dados desta empresa de sorvetes, nós temos `Clientes`, `Loja` e `Produto`, onde a interação destas 3 cria uma transação de vendas.

![](../src/img/vendas-entidades.png)

De acordo com uma das regras de modelagem DV nós temos que modelar transações como ``Links``, e para faze-lo, criaremos 3 ``hubs`` para as entidades mencionadas anteriormente e uma `Link table` para o evento de vendas, incluindo todas as colunas chave que são relevantes para os ``hubs`` e as outras colunas obrigatórias.

![](../src/img/vendas_DV.png)

Lembrando que podemos conectar `Satelites` em `Hubs` ou `Links`, porém neste cenario, estamos olhando os detalhes da transação, por tanto nossa tabela `Satelite` está conectada na transação ou `Transaction Link`.

### Vendo os dados de transação no modelo DV.

Agora, como a sorveteria é topzeira nós vamos nos tornar clientes regulares e voltar lá todos os dias e nossa simples ida irá gerar dados importantes de transação no sistema OLTP da rede de sorveterias. Imagine estes eventos:

- __Primeiro veremos como seria a primeira compra e como estes dados de compra seria distribuidos em nosso modelo:__

![](../src/img/evento-1-compra-dv.png)

O dado seria distribuido desta forma no modelo, trazendo informações descritivas para a tabela `Satelite`. Mas e se formos na sorveteria todos os dias?

__Atenção! Tabelas `Satelites` guardam histórico, entretanto ela guarda histórico quando há atualizações nas características de uma `business key` ou em relacionamentos.__

Se vamos todos os dias podemos seguir então o caminho de atualizar o registro da transação, seria mais facil, certo?

__Errado!__

Vamos deixar bem claro que __Transações não são atualizadas em `Standard link`__. E para esclarecer, pense em uma passagem de avião. Se você compra passagens ida e volta para Salvador, porque gostaria de conhecer o nordeste e depois muda de ideia e prefere iniciar a viagem em Fortaleza e finalizar em Salvador, se formos na compahia aerea eles não irão atualizar nossa passagem. O mais provável de ocorrer é, cancelarão a passagem de ida e irão gerar uma nova passagem para Fortaleza.

Então para guardar alterações, vamos precisar fazer algumas mudanças na estrutura da tabela `Link`.

### Non Historized Links

Uma das possibilidades é utilizarmos a estrutura de uma `Link` não-histórica, (em livre tradução). Como veremos abaixo uma coluna é adicionada à estrutura da `Link Table`:

| LINK_Vendas       | 
|-------------------|
| HK_Vendas_ID PK   | 
| HK_Cliente_ID FK  | 
| HK_Produto_ID FK  | 
| HK_Loja_ID FK     | 
| __Vendas_ID__     |
| LOAD_DTS          | 
| REC_SRC           | 
|                   |

Inserindo o ID da transação, nós tornamos cada transação unica, não sendo necessário armazenar histórico.

>Entretanto, armazenar dados na `Link`, como o Id de vendas, ou qualquer tipo de dado transacional aumenta a complexidade de manutenção do DV, pois transações não podem ser atualizadas.

