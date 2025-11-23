# libspech

Uma biblioteca SIP (Session Initiation Protocol) baseada em PHP para construir aplicações VoIP usando corrotinas Swoole. Esta biblioteca fornece funcionalidade de controlador de trunking para registro SIP, gerenciamento de chamadas e streaming de áudio em tempo real.

## Visão Geral

libspech é uma biblioteca SIP abrangente que permite que aplicações PHP:
- Registrar com servidores SIP
- Fazer e receber chamadas VoIP
- Lide com streaming de áudio em tempo real com suporte a vários codecs (PCMU, PCMA, G.729)
- Gerenciar estados de chamada (tocando, atender, desligar)
- Processar eventos DTMF
- Gerenciar detecção de qualidade de áudio e buffering adaptativo
- Suportar protocolos RTP/RTCP

## Requisitos

- **PHP**: 8.4+ (testado em PHP 8.4.13)
- **Extensões**:
  - Swoole (para corrotinas e rede assíncrona)
  - bcg729 (opcional, para suporte ao codec G.729)
- **Rede**: Acesso à porta UDP para SIP (padrão 5060) e portas RTP

## Instalação

### Clonar o Repositório

```bash
git clone https://github.com/berzersks/libspech.git
cd libspech
```

### Instalar Dependências

<!-- TODO: Adicionar composer.json se o Composer for usado para gerenciamento de dependências -->
Atualmente, o projeto usa um autoloader personalizado. Certifique-se de que a extensão Swoole esteja instalada:

```bash
# Install Swoole via PECL
pecl install swoole

# Or compile from source
# See: https://github.com/swoole/swoole-src
```

## Uso

### Exemplo Básico

Veja `example.php` para um exemplo completo e funcional:

```php
<?php

include 'plugins/autoloader.php';

\Swoole\Coroutine\run(function () {
    // Configure SIP credentials
    $username = 'your_sip_username';
    $password = 'your_sip_password';
    $domain = 'your_sip_domain.com';
    $host = gethostbyname($domain);

    // Create trunk controller
    $phone = new trunkController(
        $username,
        $password,
        $host,
        5060,
    );

    // Register with SIP server
    if (!$phone->register(2)) {
        throw new \Exception("Registration failed");
    }

    // Set up event handlers
    $phone->onRinging(function ($call) {
        echo "Call is ringing...\n";
    });

    $phone->onAnswer(function ($call) {
        echo "Call answered!\n";
    });

    $phone->onHangup(function ($call) {
        echo "Call ended.\n";
    });

    $phone->onReceiveAudio(function ($audioData) {
        echo strlen($audioData) . " bytes received\n";
    });

    // Make a call
    $phone->prefix = 4479;
    $phone->call('551140040104');
});
```

## Estrutura do Projeto

```
libspech/
├── plugins/                      # Core library modules
│   ├── autoloader.php           # Custom autoloader
│   ├── configInterface.json     # Autoload configuration
│   ├── Extension/               # PHP extensions and utilities
│   │   ├── plugins/             # Helper plugins (EventEmitter, DialogManager, etc.)
│   │   └── terminal/            # Terminal/shell utilities
│   ├── Packet/                  # SIP packet handlers
│   │   ├── controller/          # SIP message controllers (200, 401, 404, etc.)
│   │   └── routes/              # SIP routing logic
│   ├── Start/                   # Initialization modules
│   │   ├── console/             # Console handlers
│   │   └── events/              # Event system
│   └── Utils/                   # Utility classes
│       ├── cache/               # Caching mechanisms
│       ├── cli/                 # CLI utilities
│       ├── network/             # Network utilities
│       └── sip/                 # SIP core classes
│           ├── trunkController.php    # Main trunk controller
│           ├── softphone.php          # Softphone implementation
│           ├── rtpChannel.php         # RTP channel handler
│           ├── AudioQualityDetector.php
│           ├── AdaptiveBuffer.php
│           └── ...
├── example.php                  # Usage example
├── c.php                        # Stub generator utility
└── README.md                    # This file
```

## Scripts

### Executar Exemplo

```bash
php example.php
```

### Gerar Esqueletos de Extensão

O script `c.php` gera stubs PHP para extensões C (útil para autocompletar em IDEs):

```bash
php c.php
```

## Configuração

### Configuração do Autoloader

O autoloader é configurado via `plugins/configInterface.json`:

```json
{
  "autoload": [
    "Utils/cache",
    "Utils/cli",
    "Utils/sip",
    "Start/events",
    "Packet/controller",
    "Packet/routes",
    "Start/console",
    "Extension/plugins",
    "Extension/terminal",
    "Utils/network"
  ],
  "reloadCaseFileModify": []
}
```

### Variáveis de Ambiente

<!-- TODO: Documentar variáveis de ambiente necessárias, se houver -->
Atualmente, nenhuma variável de ambiente é necessária. A configuração é feita programaticamente.

## Codecs Suportados

A biblioteca suporta os seguintes codecs de áudio:

- **PCMU (G.711 μ-law)** - Tipo de carga útil 0
- **PCMA (G.711 A-law)** - Tipo de carga útil 8
- **G.729** - Tipo de carga útil 18 (requer a extensão bcg729)
- **telephone-event (DTMF)** - Tipo de payload 101

## Teste

<!-- TODO: Adicionar framework de testes e instruções -->
Framework de teste ainda não implementado.

## Funcionalidades

- ✅ Registro SIP com autenticação (digest MD5)
- ✅ Iniciação de chamada de saída
- ✅ Atendimento de chamadas receptivas
- ✅ Streaming de áudio RTP
- ✅ Suporte RTCP
- ✅ Suporte a múltiplos codecs
- ✅ Detecção de qualidade de áudio
- ✅ Buffer adaptativo
- ✅ Manipulação de eventos DTMF
- ✅ Transferência de chamada (REFER)
- ✅ Gerenciamento de diálogo
- ✅ Arquitetura orientada a eventos

## Limitações conhecidas

- Apenas IPv4 (suporte a IPv6 não implementado)
- Sem suporte SRTP/TLS
- Negociação limitada de SDP
- Sem transcodificação de mídia entre codecs

## Contribuindo

<!-- TODO: Adicionar diretrizes de contribuição -->
Contribuições são bem-vindas! Por favor, envie pull requests para o repositório.

## Licença

<!-- TODO: Adicionar arquivo de licença e especificar o tipo de licença -->
Informações de licença não especificadas. Por favor, contate o autor para detalhes de licenciamento.

## Créditos

Repositório: https://github.com/berzersks/libspech

---

****Nota**: Este é um projeto em desenvolvimento ativo. Algumas funcionalidades podem estar incompletas ou sujeitas a alterações.
