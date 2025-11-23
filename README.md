# libspech

**Biblioteca PHP completa para comunicação de voz real via SIP/RTP**

Uma implementação robusta do protocolo SIP (Session Initiation Protocol) com suporte total a streaming de áudio RTP/RTCP em tempo real, construída com corrotinas Swoole para alto desempenho. Esta biblioteca permite que aplicações PHP realizem chamadas VoIP reais com transmissão e recepção de áudio bidirecional.

## 🎯 Visão Geral

**libspech** é uma biblioteca SIP funcional e completa que fornece:

### 📞 Comunicação de Voz Real
- ✅ **Chamadas VoIP bidirecionais completas** - Transmissão e recepção simultânea de áudio
- ✅ **Streaming RTP em tempo real** - Envio e recebimento de pacotes de áudio via protocolo RTP
- ✅ **Suporte a múltiplos codecs** - PCMU (G.711 μ-law), PCMA (G.711 A-law), G.729
- ✅ **Monitoramento RTCP** - Controle de qualidade e sincronização de mídia
- ✅ **Conversão de codecs** - Encoding/decoding entre PCM, PCMA e PCMU

### 🔧 Funcionalidades SIP
- ✅ **Registro SIP com autenticação** - Suporte completo a MD5 Digest Authentication
- ✅ **Gerenciamento de chamadas** - INVITE, ACK, BYE, CANCEL
- ✅ **Negociação SDP** - Session Description Protocol para configuração de mídia
- ✅ **Arquitetura orientada a eventos** - Callbacks assíncronos para todos os estados de chamada

### ⚡ Características Técnicas
- **Assíncrono e não-bloqueante** - Baseado em corrotinas Swoole
- **Alta performance** - Processamento eficiente de pacotes UDP
- **Baixa latência** - Streaming direto de áudio sem buffers desnecessários
- **Código limpo e enxuto** - Apenas 11 arquivos essenciais, sem dependências complexas

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

## 📁 Estrutura do Projeto

```
libspech/
├── plugins/                          # Módulos principais da biblioteca
│   ├── autoloader.php               # Autoloader customizado
│   ├── configInterface.json         # Configuração do autoloader
│   │
│   ├── Packet/                      # 📦 Manipuladores de pacotes SIP
│   │   └── controller/
│   │       └── renderMessages.php   # Renderização de mensagens SIP/SDP
│   │
│   └── Utils/                       # 🛠️ Classes utilitárias
│       │
│       ├── cache/                   # Sistema de cache global
│       │   ├── cache.php            # Cache para estado da aplicação
│       │   └── rpcClient.php        # Cliente RPC para comunicação
│       │
│       ├── cli/                     # Interface de linha de comando
│       │   └── cli.php              # Output colorido no console
│       │
│       ├── network/                 # Utilitários de rede
│       │   └── network.php          # Resolução de IP e gerenciamento de portas
│       │
│       └── sip/                     # 🎙️ Core SIP/RTP (comunicação de voz)
│           ├── trunkController.php  # Controlador principal - Gerencia registro SIP e chamadas
│           ├── phone.php            # Gerenciamento de telefone e estados de chamada
│           ├── sip.php              # Parser e renderizador de mensagens SIP
│           ├── rtpChannels.php      # Criação e envio de pacotes RTP (áudio)
│           └── rtpc.php             # Parser de pacotes RTP/RTCP recebidos
│
├── stubs/                           # Stubs PHP para autocomplete de IDE
│   ├── bcg729Channel.php            # Interface para codec G.729
│   └── trunkController.php          # Stub da classe principal
│
├── example.php                      # 🚀 Exemplo funcional de uso
└── README.md                        # Este arquivo
```

### 🔑 Componentes Principais

| Componente | Responsabilidade | Função na Comunicação de Voz |
|------------|------------------|------------------------------|
| **trunkController.php** | Controlador SIP principal | Registro no servidor SIP, criação/gerenciamento de chamadas |
| **phone.php** | Gerenciamento de chamadas | Controle de estados (ringing, answered, hangup) |
| **sip.php** | Protocolo SIP | Parser/render de mensagens SIP e SDP |
| **rtpChannels.php** | Transmissão de áudio | Criação e envio de pacotes RTP com áudio codificado |
| **rtpc.php** | Recepção de áudio | Parsing de pacotes RTP recebidos e extração de áudio |
| **renderMessages.php** | Mensagens SIP | Renderização de respostas SIP (200 OK, ACK, etc.) |
| **network.php** | Rede | Resolução de IPs e gerenciamento de portas UDP |
| **cache.php** | Estado global | Cache de sessões e dados temporários |

## Scripts

### Executar Exemplo

```bash
php example.php
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
    "Utils/network",
    "Packet/controller"
  ],
  "reloadCaseFileModify": []
}
```

### Variáveis de Ambiente

<!-- TODO: Documentar variáveis de ambiente necessárias, se houver -->
Atualmente, nenhuma variável de ambiente é necessária. A configuração é feita programaticamente.

## 🎵 Codecs de Áudio Suportados

A biblioteca oferece suporte completo aos seguintes codecs com **conversão automática**:

| Codec | Payload Type | Taxa de Amostragem | Descrição | Status |
|-------|--------------|-------------------|-----------|---------|
| **PCMU (G.711 μ-law)** | 0 | 8000 Hz | Codec padrão para América do Norte e Japão | ✅ Completo |
| **PCMA (G.711 A-law)** | 8 | 8000 Hz | Codec padrão para Europa e resto do mundo | ✅ Completo |
| **G.729** | 18 | 8000 Hz | Codec comprimido de alta qualidade | ✅ Completo (requer bcg729) |
| **telephone-event** | 101 | 8000 Hz | Eventos DTMF (tons de teclado) | ✅ Suportado |

### 🔄 Conversão de Codecs
A biblioteca inclui funções nativas de conversão:
- `encodePcmToPcma()` / `decodePcmaToPcm()` - PCM ↔ A-law
- `encodePcmToPcmu()` / `decodePcmuToPcm()` - PCM ↔ μ-law
- `linear2alaw()` / `alaw2linear()` - Conversão linear para A-law
- `linear2ulaw()` / `ulaw2linear()` - Conversão linear para μ-law

## 🎯 Casos de Uso

Esta biblioteca é ideal para:

- 🤖 **Bots de voz automatizados** - IVR (URA), assistentes virtuais
- 📞 **Softphones em PHP** - Aplicações de telefonia integradas
- 🎙️ **Gravação de chamadas** - Captura e processamento de áudio em tempo real
- 📊 **Análise de voz** - Processamento de áudio para transcrição ou análise
- 🔗 **Integração com sistemas existentes** - Conectar aplicações PHP a infraestrutura VoIP
- 🧪 **Testes de sistemas VoIP** - Simulação de chamadas e testes automatizados

## ⚙️ Como Funciona

### Fluxo de Comunicação VoIP

```
1. REGISTRO SIP
   PHP App → [REGISTER] → Servidor SIP
   PHP App ← [401 Unauthorized] ← Servidor SIP
   PHP App → [REGISTER + Auth] → Servidor SIP
   PHP App ← [200 OK] ← Servidor SIP

2. CHAMADA SAINTE (INVITE)
   PHP App → [INVITE + SDP] → Servidor SIP → Destino
   PHP App ← [100 Trying] ← Servidor SIP
   PHP App ← [180 Ringing] ← Destino (evento: onRinging)
   PHP App ← [200 OK + SDP] ← Destino (evento: onAnswer)
   PHP App → [ACK] → Destino

3. STREAMING DE ÁUDIO RTP (Bidirecional)
   PHP App ⇄ [Pacotes RTP UDP] ⇄ Destino
   - Envio: rtpChannels.php cria pacotes RTP com áudio
   - Recepção: rtpc.php parseia pacotes RTP recebidos
   - Evento: onReceiveAudio() chamado para cada pacote recebido

4. ENCERRAMENTO (BYE)
   PHP App → [BYE] → Destino
   PHP App ← [200 OK] ← Destino (evento: onHangup)
```

## ✨ Funcionalidades

### Protocolo SIP
- ✅ Registro SIP com autenticação MD5 Digest
- ✅ Suporte a métodos: REGISTER, INVITE, ACK, BYE, CANCEL
- ✅ Parsing completo de mensagens SIP
- ✅ Negociação SDP (Session Description Protocol)
- ✅ Gerenciamento de Call-ID, tags, branches

### Mídia RTP/RTCP
- ✅ **Transmissão RTP** - Envio de pacotes de áudio em tempo real
- ✅ **Recepção RTP** - Parsing e decodificação de áudio recebido
- ✅ **RTCP** - Relatórios de qualidade e sincronização
- ✅ **Múltiplos codecs** - PCMU, PCMA, G.729
- ✅ **Conversão de codecs** - Encoding/decoding automático

### Eventos e Callbacks
- ✅ `onRinging()` - Chamada tocando no destino
- ✅ `onAnswer()` - Chamada atendida
- ✅ `onHangup()` - Chamada encerrada
- ✅ `onReceiveAudio($audioData)` - Áudio recebido em tempo real

## 🚧 Limitações Conhecidas

- ⚠️ **Apenas IPv4** - Suporte a IPv6 não implementado
- ⚠️ **Sem SRTP/TLS** - Comunicação não criptografada
- ⚠️ **Chamadas de saída apenas** - Recepção de chamadas (servidor) não implementada nesta versão
- ⚠️ **Um codec por chamada** - Sem transcodificação dinâmica durante a chamada

## 🧪 Testes

<!-- TODO: Adicionar framework de testes e instruções -->
Framework de teste ainda não implementado. Para testar:

```bash
# Execute o exemplo básico com suas credenciais SIP
php example.php
```

## 🤝 Contribuindo

Contribuições são bem-vindas! Para contribuir:

1. Fork o repositório
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'Add: Nova funcionalidade'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

## 📄 Licença

<!-- TODO: Adicionar arquivo de licença e especificar o tipo de licença -->
Informações de licença não especificadas. Por favor, contate o autor para detalhes de licenciamento.

## 🏆 Créditos

**Repositório**: https://github.com/berzersks/libspech

---

## 💡 Notas Importantes

### Sobre Comunicação de Voz Real

Esta biblioteca implementa **comunicação de voz bidirecional completa**:

- ✅ **Não é apenas sinalização** - Implementa tanto SIP (sinalização) quanto RTP (mídia)
- ✅ **Áudio real em tempo real** - Transmite e recebe pacotes de áudio via UDP
- ✅ **Pronto para produção** - Testado com servidores SIP reais (Asterisk, FreeSWITCH, etc.)
- ✅ **Baixa latência** - Processamento assíncrono com Swoole para performance máxima

### Diferencial

Ao contrário de muitas bibliotecas SIP em PHP que apenas gerenciam sinalização, **libspech** é uma solução completa que:
- Registra e autentica com servidores SIP
- Negocia parâmetros de mídia via SDP
- **Transmite e recebe áudio real via RTP**
- Suporta múltiplos codecs com conversão automática
- Processa eventos em tempo real

---

**Status do Projeto**: ✅ Funcional e em desenvolvimento ativo

Para dúvidas, sugestões ou reportar problemas, abra uma [issue no GitHub](https://github.com/berzersks/libspech/issues).
