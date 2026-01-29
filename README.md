# teste_backend
teste pratico desenvolvedor backend node
Objetivo

Desenvolver uma API Backend responsável por receber pedidos de um Marketplace via Webhook, validar e transformar os dados e integrar com um ERP simulado, garantindo idempotência, tratamento de erros e resiliência.

🧱 Stack Obrigatória

Node.js + TypeScript

Express ou Fastify

MongoDB ou MySQL/Postgress

🚀 Execução Local
npm install
npm run dev

Endpoints mínimos

POST /webhook/orders → Recebe pedidos do Marketplace

POST /erp/orders → Mock do ERP

📥 Webhook de Entrada (Marketplace)
Endpoint

POST /webhook/orders

Payload recebido
{
  "event": "order.created",
  "marketplace": "MERCADO_LIVRE",
  "order": {
    "id": "MLB-987654321",
    "created_at": "2026-01-22T12:45:30.000Z",
    "total_amount": 259.9,
    "currency": "BRL",
    "payment": {
      "method": "credit_card",
      "installments": 3,
      "paid": true
    },
    "customer": {
      "id": "CUST-445566",
      "name": "Duda Coan",
      "email": "coan.eduarda@email.com",
      "document": {
        "type": "CPF",
        "number": "123.456.789-09"
      },
      "phone": "+55 (47) 9 9999-8888"
    },
    "shipping": {
      "address": {
        "street": "Rua das Flores",
        "number": "123",
        "complement": "Apto 302",
        "district": "Centro",
        "city": "Blumenau",
        "state": "SC",
        "zip_code": "89010-000",
        "country": "BR"
      },
      "estimated_delivery": "2026-01-28"
    },
    "items": [
      {
        "sku": "CAMISA-BRASIL-2026",
        "name": "Camisa Brasil Copa 2026",
        "quantity": 2,
        "unit_price": 129.95
      }
    ]
  }
}

✅ Campos Obrigatórios e Validações
Campos obrigatórios

event

marketplace

order.id

order.created_at

order.total_amount

order.currency

order.payment.method

order.payment.paid

order.customer.name

order.customer.email

order.customer.document.type

order.customer.document.number

order.shipping.address.*

order.items[] (mínimo 1 item)

Validações obrigatórias

quantity > 0

unit_price > 0

Soma dos itens deve bater com total_amount

Pedido duplicado deve ser tratado por idempotência

❌ Formato de Erro (Payload Inválido)
HTTP 400 – Bad Request
{
  "error": "INVALID_PAYLOAD",
  "message": "Campos obrigatórios ausentes ou inválidos",
  "fields": [
    "order.id",
    "order.customer.document.number",
    "order.items[0].quantity"
  ]
}


Se o payload for inválido:

Não salvar no banco

Não enviar ao ERP

🔄 Transformação de Dados (Mapping)
Marketplace	ERP	Regra

order.id	codigo_externo	Direto

created_at	data_pedido	ISO → DD/MM/YYYY

customer.name	razao_social	Uppercase

document.number	cpf_cnpj	Remover máscara

phone	telefone	Remover símbolos

zip_code	cep	Remover hífen

country	pais	BR → BRASIL

payment.method	forma	Enum

payment.paid	status	true → PAGO

items[].quantity * unit_price	valor_total	Cálculo

📤 Payload Enviado para o ERP (Formato Esperado)
{
  "pedido": {
    "codigo_externo": "MLB-987654321",
    "data_pedido": "22/01/2026",
    "cliente": {
      "razao_social": "DUDA COAN",
      "email": "coan.eduarda@email.com",
      "cpf_cnpj": "12345678909",
      "telefone": "47999998888"
    },
    "endereco_entrega": {
      "logradouro": "Rua das Flores",
      "numero": "123",
      "complemento": "Apto 302",
      "bairro": "Centro",
      "cidade": "Blumenau",
      "uf": "SC",
      "cep": "89010000",
      "pais": "BRASIL"
    },
    "itens": [
      {
        "codigo_produto": "CAMISA-BRASIL-2026",
        "descricao": "Camisa Brasil Copa 2026",
        "quantidade": 2,
        "valor_unitario": 129.95,
        "valor_total": 259.9
      }
    ],
    "valores": {
      "total_produtos": 259.9,
      "frete": 0,
      "total_pedido": 259.9
    },
    "pagamento": {
      "forma": "CARTAO_CREDITO",
      "parcelas": 3,
      "status": "PAGO"
    }
  }
}

🧠 Regras de Negócio

Idempotência: usar codigo_externo como chave única

Pedido duplicado:

Retornar 200 OK

Não reenviar ao ERP

Pedido não pago:

Salvar como pendente

Não enviar ao ERP

🏗 Simulação do ERP

O ERP deve simular:

Sucesso

Erro (HTTP 500)

Timeout (ex: 10s)

🔁 Resiliência

Implementar retry automático (mín. 3 tentativas)

Processamento do ERP deve ocorrer em background

Webhook deve responder em até 2 segundos
