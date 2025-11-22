# Tutorial Passo a Passo: Integrando WhatsApp Cloud API com Laravel para Mapeamento de Dados em CRM

## Introdução

Este tutorial foi desenhado para guiar desenvolvedores, passo a passo, na criação de uma integração robusta entre a WhatsApp Business Cloud API e uma aplicação Laravel. O objetivo é construir uma base sólida que permita não apenas enviar e receber mensagens, mas também armazenar o histórico de conversas em um banco de dados. Essa estrutura é o primeiro e mais crucial passo para um futuro mapeamento de dados com qualquer sistema de CRM. Para simplificar a comunicação com a API da Meta, utilizaremos a biblioteca `missael-anda/laravel-whatsapp`, que abstrai grande parte da complexidade do processo.

---

## 1. Pré-requisitos e Configuração na Plataforma Meta

Antes de escrever qualquer linha de código, é fundamental preparar o ambiente na plataforma da Meta e obter as credenciais de autenticação.

### 1.1. Visão Geral da WhatsApp Cloud API

A WhatsApp Cloud API é uma solução projetada para empresas que precisam de escala, automação e integração. Diferente do aplicativo WhatsApp Business, que é ideal para pequenos negócios com baixo volume de mensagens, a API permite a conexão com sistemas externos, como CRMs e plataformas de e-commerce. Ela foi feita para múltiplos atendentes, automação completa e envio de mensagens em larga escala, sendo a escolha ideal para mapear a jornada do cliente de ponta a ponta.

### 1.2. Configuração da Aplicação na Meta

Para interagir com a API, você precisará de credenciais essenciais. Siga os passos abaixo para obtê-las:

1. **Acesse a Central do Desenvolvedor**: Faça login na Central do Desenvolvedor da Meta com sua conta.
2. **Crie um Novo Aplicativo**: Clique em "Meus aplicativos" e crie um novo aplicativo. Selecione o tipo "Business" (Negócios).
3. **Configure o Produto WhatsApp**: No painel do seu novo aplicativo, encontre o produto "WhatsApp" e clique em "Configurar".
4. **Obtenha as Credenciais**: Você será redirecionado para a página "Configuração da API", onde encontrará as informações necessárias:
   * **Token de Acesso Temporário**: Um token que expira em 24 horas será exibido. Para produção, será necessário gerar um token de acesso permanente.
   * **ID do Número de Telefone**: Um número de telefone de teste é fornecido por padrão. O ID associado a ele está listado logo abaixo do token de acesso.

Guarde essas duas credenciais (Token de Acesso e ID do Número de Telefone), pois elas serão utilizadas para configurar sua aplicação Laravel.

Com as chaves de acesso em mãos, estamos prontos para configurar o ambiente de desenvolvimento e conectar nossa aplicação Laravel à API da Meta.

---

## 2. Configurando o Ambiente Laravel

Agora, vamos instalar e configurar a biblioteca que facilitará a comunicação com a API do WhatsApp.

### 2.1. Instalação da Biblioteca

Abra o terminal na raiz do seu projeto Laravel e execute o seguinte comando Composer para instalar o pacote:

```bash
composer require missael-anda/laravel-whatsapp
```

### 2.2. Publicação do Arquivo de Configuração

Para ter maior controle sobre as configurações, publique o arquivo de configuração do pacote. Isso criará o arquivo `config/whatsapp.php`.

```bash
php artisan vendor:publish --provider="MissaelAnda\Whatsapp\WhatsappServiceProvider" --tag=config
```

### 2.3. Configuração das Variáveis de Ambiente

Abra o arquivo `.env` do seu projeto e adicione as seguintes variáveis. Preencha os valores com as credenciais que você obteve na plataforma da Meta. O `WHATSAPP_WEBHOOK_VERIFY_TOKEN` é uma senha criada por você para proteger seu webhook.

| Variável de Ambiente | Descrição | Exemplo |
|---------------------|-----------|---------|
| `WHATSAPP_TOKEN` | O token de acesso (temporário ou permanente) gerado na Central do Desenvolvedor. | `EAAJB...` |
| `WHATSAPP_NUMBER_ID` | O ID do número de telefone que enviará e receberá as mensagens. | `106540352242922` |
| `WHATSAPP_WEBHOOK_VERIFY_TOKEN` | Um token secreto criado por você para a verificação do webhook (a ser usado na Seção 4). | `senha-super-secreta-123` |

Com o ambiente configurado, sua aplicação Laravel já está apta a se comunicar com a API do WhatsApp. O próximo passo é enviar nossa primeira mensagem.

---

## 3. Enviando Mensagens via API

O envio de mensagens é controlado por regras específicas da Meta, principalmente relacionadas à janela de atendimento de 24 horas e ao uso de templates.

### 3.1. Entendendo a Janela de Atendimento de 24 Horas

A Meta incentiva conversas de alta qualidade e que são iniciadas pelo cliente. Por isso, existe a "janela de atendimento de 24 horas". Essa janela começa a contar a partir da última mensagem enviada pelo cliente. Dentro desse período, sua empresa pode enviar mensagens de formato livre (texto, mídia, etc.) sem custo adicional e sem a necessidade de usar templates. Se o cliente não responder em 24 horas, a janela se fecha. Para iniciar uma nova conversa ou retomar o contato, você obrigatoriamente precisará usar uma mensagem de template.

### 3.2. As Categorias de Mensagens de Template

A Meta classifica as conversas iniciadas pela empresa em três categorias, cada uma com finalidades e custos distintos. Utilizar a categoria correta é crucial para evitar rejeições e otimizar os custos.

* **Marketing**: Para promoções, lançamentos e campanhas. Requer consentimento explícito (opt-in) do cliente.
* **Utilitárias**: Para confirmações de pedidos, alertas de entrega, lembretes e atualizações de status. Focadas em facilitar a experiência do cliente.
* **Autenticação**: Para envio de senhas de uso único (OTP) e verificação de identidade.

### 3.3. Enviando Mensagens de Template

Como mencionado, ao iniciar uma conversa ou responder a um cliente fora da janela de 24 horas, é obrigatório o uso de uma Mensagem de Template previamente aprovada pela Meta. A biblioteca simplifica este envio.

Veja um exemplo de como enviar uma mensagem de template que possui um parâmetro no corpo do texto:

```php
use MissaelAnda\Whatsapp\Facades\Whatsapp;
use MissaelAnda\Whatsapp\Messages\Components\Parameters\Text;
use MissaelAnda\Whatsapp\Messages\TemplateMessage;

Whatsapp::send('DESTINATION_PHONE_NUMBER', // Ex: 5511999998888
    TemplateMessage::create()
        ->name('nome_do_seu_template')
        ->language('pt_BR')
        ->body([
            Text::create('Valor do Parâmetro 1'),
        ])
);
```

**Nota**: Substitua `DESTINATION_PHONE_NUMBER` e `nome_do_seu_template` pelos valores corretos.

### 3.4. Como Criar e Gerenciar Templates

Para criar novos templates, você deve acessar o **WhatsApp Manager** no painel da sua conta de negócios da Meta. Lá, você poderá criar mensagens com texto, mídia e variáveis, e submetê-las para aprovação.

Após dominar o envio, o próximo passo lógico é preparar a aplicação para receber e processar as respostas dos usuários.

---

## 4. Recebendo Mensagens com Webhooks

Para receber mensagens e notificações em tempo real, utilizamos Webhooks.

### 4.1. O que são Webhooks?

Webhooks são notificações automáticas via HTTP que a plataforma da Meta envia para a sua aplicação sempre que um evento ocorre. Por exemplo, quando um cliente envia uma mensagem para o seu número, a Meta envia um POST request para uma URL que você define, contendo todos os dados da mensagem em formato JSON.

### 4.2. Expondo sua Aplicação Local com ngrok

Durante o desenvolvimento, sua aplicação Laravel roda em um ambiente local (ex: `localhost:8000`), que não é acessível pela internet. Para que a Meta possa enviar notificações de webhook para sua máquina, você precisa de uma URL pública. Ferramentas como o **ngrok** criam um túnel seguro para sua aplicação local.

1. Instale o ngrok (siga as instruções no site oficial).
2. Inicie seu servidor de desenvolvimento Laravel (`php artisan serve`).
3. Em um novo terminal, execute o seguinte comando para expor a porta 8000:

```bash
ngrok http 8000
```

O ngrok gerará uma URL pública (ex: `https://aleatorio.ngrok-free.app`). Esta será a sua URL de Callback temporária.

### 4.3. Configuração do Webhook na Meta

Siga estes passos no painel do seu aplicativo na Meta para configurar o recebimento de mensagens:

1. No menu lateral, vá para **WhatsApp > Configuração da API**.
2. Na seção de **Webhooks**, clique em **Editar**.
3. **URL de Callback**: Insira a URL pública gerada pelo ngrok, seguida pelo endpoint padrão do pacote: `https://aleatorio.ngrok-free.app/whatsapp/webhook`.
4. **Token de Verificação**: Preencha este campo com o mesmo valor que você definiu na variável `WHATSAPP_WEBHOOK_VERIFY_TOKEN` do seu arquivo `.env`.
5. **Assinar Eventos**: Clique em "Gerenciar" e assine (subscribe) o campo `messages` para receber notificações de novas mensagens.

### 4.4. Processando a Notificação no Laravel

A biblioteca `missael-anda/laravel-whatsapp` já cuida de todo o processo de verificação do webhook. Quando uma notificação de mensagem válida é recebida, ela dispara automaticamente o evento `MissaelAnda\Whatsapp\Events\MessagesReceived`.

Abaixo, um exemplo da estrutura JSON (payload) que sua aplicação receberá para uma mensagem de texto:

```json
{
  "object": "whatsapp_business_account",
  "entry": [{
    "id": "WHATSAPP_BUSINESS_ACCOUNT_ID",
    "changes": [{
      "value": {
        "messaging_product": "whatsapp",
        "metadata": {
          "display_phone_number": "PHONE_NUMBER",
          "phone_number_id": "PHONE_NUMBER_ID"
        },
        "contacts": [{
          "profile": { "name": "NAME" },
          "wa_id": "PHONE_NUMBER"
        }],
        "messages": [{
          "from": "PHONE_NUMBER",
          "id": "wamid.ID",
          "timestamp": "TIMESTAMP",
          "text": { "body": "MESSAGE_BODY" },
          "type": "text"
        }]
      },
      "field": "messages"
    }]
  }]
}
```

Observe o campo `id` dentro do objeto `messages` (com o prefixo `wamid`). Este é o identificador único da mensagem do WhatsApp, que será fundamental na próxima seção para garantir que cada mensagem seja salva em nosso banco de dados apenas uma vez.

---

## 5. Mapeando e Armazenando Conversas no Banco de Dados

Agora que nossa aplicação recebe notificações via webhooks, vamos conectar essa funcionalidade ao núcleo do Laravel. A biblioteca `missael-anda/laravel-whatsapp` convenientemente dispara um evento (`MessagesReceived`) para cada nova mensagem, permitindo-nos criar Listeners para executar ações, como salvar os dados, de forma desacoplada e organizada.

### 5.1. Criação da Estrutura do Banco de Dados

Vamos criar uma migration no Laravel para a tabela que armazenará as mensagens. Execute o comando:

```bash
php artisan make:migration create_whatsapp_messages_table
```

Agora, edite o método `up()` no arquivo da migration gerada para definir a estrutura da tabela:

```php
// database/migrations/xxxx_xx_xx_xxxxxx_create_whatsapp_messages_table.php

public function up(): void
{
    Schema::create('whatsapp_messages', function (Blueprint $table) {
        $table->id();
        $table->string('wamid')->unique(); // ID da mensagem do WhatsApp
        $table->string('from_phone');
        $table->text('body');
        $table->string('timestamp');
        $table->enum('direction', ['in', 'out']); // 'in' para recebidas, 'out' para enviadas
        $table->timestamps();
    });
}
```

Execute a migration com `php artisan migrate`.

### 5.2. Criando um Listener para Salvar Mensagens

Vamos criar um Listener que "ouve" o evento `MessagesReceived` e salva os dados no banco.

```bash
php artisan make:listener StoreIncomingWhatsappMessage
```

No arquivo do Listener (`app/Listeners/StoreIncomingWhatsappMessage.php`), adicione a lógica para processar o evento no método `handle`:

```php
// app/Listeners/StoreIncomingWhatsappMessage.php

use App\Models\WhatsappMessage; // Crie este model com `php artisan make:model WhatsappMessage`
use Illuminate\Support\Arr;
use MissaelAnda\Whatsapp\Events\MessagesReceived;

class StoreIncomingWhatsappMessage
{
    public function handle(MessagesReceived $event): void
    {
        // O payload vem aninhado, extraímos o array de mensagens
        $messages = Arr::get($event->data, 'entry.0.changes.0.value.messages', []);

        foreach ($messages as $message) {
            // Este tutorial foca apenas em mensagens de texto para simplicidade.
            if ($message['type'] !== 'text') {
                continue;
            }

            // Verifica se a mensagem com este 'wamid' já existe para evitar duplicatas.
            $existingMessage = WhatsappMessage::where('wamid', $message['id'])->first();

            if (!$existingMessage) {
                // Esta verificação é crucial. Em sistemas distribuídos, é possível que um webhook
                // seja entregue mais de uma vez. Usar o `wamid` como um identificador único
                // garante a idempotência da nossa operação, prevenindo a duplicação de dados.
                WhatsappMessage::create([
                    'wamid' => $message['id'],
                    'from_phone' => $message['from'],
                    'body' => $message['text']['body'],
                    'timestamp' => $message['timestamp'],
                    'direction' => 'in', // Mensagem recebida
                ]);
            }
        }
    }
}
```

### 5.3. Registrando o Evento e o Listener

Para que o listener seja ativado pelo evento, você precisa registrá-lo no `EventServiceProvider`. Abra o arquivo `app/Providers/EventServiceProvider.php` e adicione o mapeamento ao array `$listen`.

```php
// app/Providers/EventServiceProvider.php

protected $listen = [
    \MissaelAnda\Whatsapp\Events\MessagesReceived::class => [
        \App\Listeners\StoreIncomingWhatsappMessage::class,
    ],
    // ... outros eventos
];
```

Com a lógica de armazenamento implementada, é vital garantir que as rotas da sua aplicação estejam devidamente protegidas.

---

Para manter a saúde e a reputação do seu número de WhatsApp, siga rigorosamente as políticas da Meta. As práticas mais críticas são:

* **Opt-in Claro**: Nunca envie mensagens para um cliente que não tenha autorizado explicitamente o contato. Listas de contatos compradas são proibidas e levam a bloqueios.
* **Personalização**: Utilize os dados que você possui para personalizar as mensagens. Chamar o cliente pelo nome ou referenciar uma compra anterior aumenta a relevância e o engajamento.
* **Monitoramento de Métricas**: Acompanhe constantemente as métricas de qualidade do seu número no WhatsApp Manager. Taxas altas de bloqueio e denúncias podem levar à redução de limites de envio ou até ao bloqueio do número.
