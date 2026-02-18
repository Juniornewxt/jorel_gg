# Jorel GG (Experimental)

Este é um projeto pessoal em fase inicial de desenvolvimento, **feito utilizando Go (Golang) e Wails**. O Jorel é um assistente virtual capaz de executar tarefas no computador, interagir com APIs e processar documentos usando IA (Gemini, OpenAI, Ollama ou LlamaCPP).

Funciona um pouco como um "agente", ou seja, ele não apenas responde a perguntas, mas pode usar ferramentas para mexer no sistema operacional, criar arquivos, monitorar logs e conectar com outros serviços via MCP (Model Context Protocol).

## Como Configurar (config.json)

O arquivo `config.json` controla tudo. Se não existir, o Jorel cria um padrão ao iniciar.

### Parâmetros Principais

*   **provedor_selecionado**: Qual IA usar (`gemini`, `openai`, `ollama` ou `llamacpp`).
*   **usar_mcp**: `true` para ativar o suporte a servidores MCP, `false` para desativar.
*   **quantidade_msg_lembrar**: Quantas mensagens anteriores o Jorel mantém na memória de curto prazo (contexto).
*   **modo_somente_chat**: Se `true`, desativa todas as ferramentas (segurança máxima). O Jorel vira apenas um chat.
*   **memorizar_documento_vdb**: Se `true`, permite que documentos carregados no botão "Memorizar Documento" sejam vetorizados (RAG) para perguntas futuras.
*   **nivel_seguranca**: `0` (sem proteção), `1` (protege o básico), `2` (proteção avançada), `3` (pergunta antes de rodar comandos). *Nota: Ainda em implementação.*

### Configuração das IAs

*   **gemini**:
    *   `api_key`: Sua chave da Google AI Studio.
    *   `modelo`: Ex: `gemini-2.0-flash-exp`.
*   **openai**:
    *   `api_key`: Sua chave da OpenAI.
    *   `modelo`: Ex: `gpt-4o`.
*   **ollama**:
    *   `modelo`: Nome do modelo local (ex: `llama3`).
    *   `url`: Endereço do Ollama (padrão `http://localhost:11434`).
*   **llamacpp**:
    *   `modelo`: Caminho absoluto para o arquivo `.gguf`.
    *   `gpu_layers`: Quantas camadas rodar na GPU (ex: `33` para tudo, `0` para CPU).

### Integrações e Ferramentas

*   **mcp_servers**: Lista de servidores MCP para conectar. Exemplo:
    ```json
    "filesystem": {
      "comando": "npx",
      "argumentos": ["-y", "@modelcontextprotocol/server-filesystem", "C:/Users/voce/Documents"]
    }
    ```
*   **apis_permitidas**: Atalhos para APIs externas.
*   **comandos_bloqueados**: Lista de comandos de terminal que o Jorel é proibido de executar (ex: `rm -rf`, `format`).
*   **servidor_api**: Configura a API REST que o próprio Jorel expõe (porta, token).

## Segurança e Privacidade

1.  **Chaves de API Criptografadas**: Assim que você salva suas chaves (OpenAI, Gemini, Telegram) no `config.json` e inicia o Jorel, ele automaticamente criptografa esses valores. Se você abrir o arquivo depois, verá algo ilegível como `jorelk-a1b2c3...`. Isso impede que malwares simples roubem suas chaves.
2.  **Banco de Dados WhatsApp**: O arquivo local `whatsapp.db` (onde ficam as sessões) também é criptografado.
3.  **Whitelist de Usuários**: Tanto no Telegram quanto no WhatsApp, você define explicitamente quem pode falar com o Jorel (`usuario_permitido` ou `id_telefone_utilizado`). Qualquer outra pessoa será ignorada.

## Configurando Mensageiros (Controle Remoto)

Você pode controlar o Jorel pelo celular via Telegram ou WhatsApp, executar comandos, pedir que envie mensagens, fotos, documentos, etc do seu pc.

### Telegram
1.  Abra o Telegram e fale com o **@BotFather**.
2.  Envie `/newbot` e siga os passos para criar um bot.
3.  Ele vai te dar um **Token** (ex: `123456:ABC-DEF1234gh...`). Coloque isso no `config.json` em `telegram.token`.
4.  Para saber seu ID de usuário, fale com o bot **@userinfobot**. Ele vai te dar um número (ex: `123456789`).
5.  Coloque esse número em `telegram.usuario_permitido`. Assim, só VOCÊ consegue comandar o bot.

### WhatsApp
1.  Configure `whatsapp.habilitado: true` no `config.json`.
2.  Ao iniciar o Jorel, clique no ícone "QR Code" (se aparecer) ou verifique o terminal para escanear o código com seu celular (Menu > Aparelhos Conectados > Conectar).
3.  **Segurança**: Para garantir que só você fale com o Jorel (e não qualquer grupo que ele entre), você precisa pegar o ID do seu telefone.
    *   Mande uma mensagem para o Jorel pelo WhatsApp.
    *   Olhe o log do Jorel "Não esqueça de habilitar debug para mostrar o log". Vai aparecer algo como: `⚠️ Mensagem ignorada de: 551199999999@s.whatsapp.net`.
    *   Copie esse ID (ex: `551199999999` ou apenas os números, conforme o log mostrar).
    *   Cole no `config.json` em `whatsapp.id_telefone_utilizado`.
4.  Reinicie o Jorel. Agora ele só responderá a esse número.

## Personalizando o Jorel (Prompt)

Você pode "ensinar" o Jorel a se comportar de maneira específica ou dar instruções fixas criando o arquivo `prompt.txt` na mesma pasta do executável.

Tudo o que você escrever ali será usado como a "personalidade" dele ou regras de negócio.
Exemplo de prompt:
> "Você é um especialista em Python. Sempre que eu pedir código, use type hints. Se eu perguntar sobre clima, consulte a API 'weather' automaticamente."

Se o arquivo não existir, o Jorel usa um prompt interno padrão focado em ser um assistente prestativo e conciso.

## O que o Jorel pode fazer?

Aqui estão alguns exemplos de comandos que você pode pedir para ele:

**Sistema & Arquivos**
*   "Crie uma pasta chamada 'Relatorios' na área de trabalho."
*   "Abra o bloco de notas e escreva 'Lembrar de comprar leite'."
*   "Liste os arquivos da pasta Documents e me diga quais são PDFs."
*   "Tire um print da tela agora."

**Automação & Office**
*   "Crie uma planilha Excel chamada 'vendas.xlsx' com colunas Data, Produto e Valor, e adicione 3 exemplos."
*   "Gere um PDF com um resumo sobre buracos negros."
*   "Crie uma apresentação PowerPoint sobre o projeto Jorel com 3 slides."
*   "Execute as ações conforme descrito dentro do arquivo xxx ou em uma URL."

**Monitoramento**
*   "Monitore o processo 'notepad.exe' e me avise se ele for fechado."
*   "Vigie o arquivo 'erro.log' a cada 5 segundos e me alerte se aparecer a palavra 'Error'."

**Web & Download**
*   "Pesquise na web sobre o preço do Bitcoin hoje."
*   "Baixe a imagem dessa URL: [link] e salve como foto.jpg."
*   "Leia o conteúdo desse site: [link] e faça um resumo."

**RAG (Conversar com Documentos)**
*   Selecione um PDF ou TXT clicando no botão "Memorizar Documento". Ele vai "memorizar" o conteúdo.
*   Depois pergunte: "O que o documento fala sobre prazos?"

**APIs e MCP (Interação com serviços externos)**
*   Você pode adicionar a URL de alguma API que o Jorel pode acessar. Ensine ele no prompt, por exemplo: "se o usuário quiser saber sobre piada, vá para 'piada': path_parameters='random_joke'".
*   Conecte a servidores MCP do docker ou crie seus próprios: "Use sua imaginação, talvez um servidor MCP para conectar via SSH e realizar comandos, ou usar servidores existentes para coletar dados de APIs, etc."

## ⚠️ Aviso Importante

Modelos de IA podem cometer erros (**alucinações**), tenha cuidado.
Algumas coisas podem não funcionar por vários motivos, como prompt ruim ou modelos menos poderosos. Esteja ciente disso ao executar comandos críticos ou confiar cegamente nas respostas.
Este projeto por enquento só tem versão para Windows, mas estou trabalhando em uma versão para Linux e MacOS.

## 🤝 Contribuições e Doações

Eu não consegui testar o **OpenAI** 100%, pois não tenho mais créditos de API (chaves). Pode ser que algo tenha quebrado.
*   **Encontrou um problema?** Avise se tiver problema.
*   **Quer ajudar?** Caso queira doar chaves para que eu possa testar e implementar novos provedores e funcionalidades, ficarei grato.
*   **Sugestões?** Se tiver sugestão de novas ferramentas ficarei feliz em receber e tentar implementar!
