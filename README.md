# 🔥 Miniguia de Estudos: Firewall — Caderno Temático no NotebookLM

> Projeto desenvolvido para o desafio da **DIO** sobre uso de IA como ferramenta de **aprendizagem ativa**: curadoria de fontes, engenharia de prompts e organização do conhecimento em um caderno temático no [NotebookLM](https://notebooklm.google.com).

![Tema](https://img.shields.io/badge/tema-Firewall-red)
![Ferramenta](https://img.shields.io/badge/ferramenta-NotebookLM-blue)
![Status](https://img.shields.io/badge/status-concluído-success)

---

## 📑 Sumário

- [1. Contexto e Objetivos](#1-contexto-e-objetivos)
- [2. Curadoria de Fontes](#2-curadoria-de-fontes)
- [3. Engenharia de Prompts e "Cicatrizes"](#3-engenharia-de-prompts-e-cicatrizes)
- [4. Miniguia de Estudo](#4-miniguia-de-estudo)
  - [4.1 Resumos estruturados](#41-resumos-estruturados)
  - [4.2 Glossário](#42-glossário)
  - [4.3 Prompts reutilizáveis](#43-prompts-reutilizáveis)
- [5. Como reproduzir este caderno](#5-como-reproduzir-este-caderno)
- [6. Aprendizados e próximos passos](#6-aprendizados-e-próximos-passos)

---

## 1. Contexto e Objetivos

### Por que Firewall?

Atuo na área de TI com suporte técnico, infraestrutura e automação. Firewall é um daqueles temas que **todo mundo acha que sabe** — "é o que bloqueia porta" — mas que, na prática, é onde nascem boa parte dos incidentes de segurança: regra mal ordenada, regra órfã que ninguém remove, `any → any` esquecido em produção, confusão entre NAT e filtragem.

Escolhi o tema para sair do conhecimento superficial e construir uma base conceitual sólida, ligando **teoria** (tipos, arquiteturas, políticas) a **prática** (regras reais em iptables/nftables e pfSense).

### Objetivos de estudo

| # | Objetivo | Como sei que atingi |
|---|----------|---------------------|
| 1 | Diferenciar com clareza as gerações de firewall (filtro de pacotes, stateful, proxy, NGFW, WAF) | Consigo explicar em qual camada OSI cada um atua e o que cada geração resolveu da anterior |
| 2 | Entender a anatomia de uma regra e a lógica de avaliação | Consigo ler um ruleset e prever qual regra vai casar com um pacote |
| 3 | Dominar o conceito de política *default deny* e filtragem de egresso | Consigo justificar tecnicamente por que bloquear saída também importa |
| 4 | Reconhecer arquiteturas de rede (DMZ, bastion host, segmentação) | Consigo desenhar um diagrama de topologia com zonas de confiança |
| 5 | Saber o que um firewall **não** resolve | Consigo listar limitações e o papel do firewall dentro de defesa em profundidade |

### Escopo definido (e o que ficou de fora)

✅ **Dentro:** conceitos, tipos, arquiteturas, política de regras, operação e limitações.
❌ **Fora:** configuração vendor-specific avançada (Fortinet/Palo Alto), tuning de performance, inspeção TLS em profundidade.

Delimitar escopo foi essencial — nas primeiras tentativas o NotebookLM devolvia respostas dispersas justamente porque a pergunta era ampla demais.

---

## 2. Curadoria de Fontes

Critérios de seleção: **acesso aberto**, **autoridade técnica**, **complementaridade** (teoria + prática + vocabulário em PT-BR) e **atualidade complementar** (a base normativa é mais antiga, então adicionei fontes de documentação viva).

| # | Fonte | Tipo | Idioma | Por que entrou no caderno |
|---|-------|------|--------|---------------------------|
| 1 | **NIST SP 800-41 Rev. 1 — Guidelines on Firewalls and Firewall Policy** — [página](https://csrc.nist.gov/pubs/sp/800/41/r1/final) · [PDF](https://tsapps.nist.gov/publication/get_pdf.cfm?pub_id=901083) | PDF / norma | EN | Base conceitual canônica: tipos de firewall, arquiteturas, política e ciclo de vida. É a espinha dorsal do caderno |
| 2 | **pfSense Documentation — Firewall** ([docs.netgate.com](https://docs.netgate.com/pfsense/en/latest/firewall/index.html)) | Documentação web | EN | Tradução da teoria para prática: ordem das regras, ingress/egress, block vs reject, aliases, boas práticas |
| 3 | **Linux Packet Filtering HOWTO (netfilter/iptables)** ([iptables.org](https://iptables.org/documentation/HOWTO/packet-filtering-HOWTO.html)) | Texto/HOWTO | EN | Mostra o firewall "por dentro": chains, hooks, match de estado, targets. Fonte de exemplos de comando |
| 4 | **Cartilha de Segurança para Internet — CERT.br / CGI.br** ([cartilha.cert.br](https://cartilha.cert.br/)) | PDF / fascículos | PT-BR | Vocabulário em português e visão de firewall pessoal/doméstico. Serviu para checar tradução de termos |
| 5 | **pfSense — Basic Firewall Configuration Example** ([docs.netgate.com](https://docs.netgate.com/pfsense/en/latest/recipes/example-basic-configuration.html)) | Documentação web | EN | Estudo de caso concreto de lockdown de LAN/DMZ, útil para exercícios de aplicação |

> 💡 **Nota de curadoria:** o NIST SP 800-41 Rev. 1 é de 2009. Ele é excelente para fundamentos, mas **não cobre bem** NGFW moderno, firewall em nuvem (security groups) e microsegmentação. Isso virou uma limitação explícita do caderno — e uma das "cicatrizes" documentadas na seção 3.

### Onde ficam os arquivos

```
/fontes
  ├── nist-sp-800-41r1.pdf
  ├── cartilha-certbr-fasciculo.pdf
  └── links.md          # URLs das fontes consultadas via web no NotebookLM
```

---

## 3. Engenharia de Prompts e "Cicatrizes"

Esta é a seção mais importante do repositório: o **raciocínio** por trás do resultado, não só o resultado.

### 3.1 Perguntas estratégicas elaboradas

Organizei as perguntas em três níveis, seguindo uma escada de profundidade:

**Nível 1 — Mapeamento (o que existe?)**
1. Quais tipos de firewall são descritos nas fontes e em qual camada do modelo OSI cada um atua?
2. Quais arquiteturas de implantação são recomendadas e em que cenário cada uma se aplica?

**Nível 2 — Relação (como se conectam?)**
3. Qual problema o firewall *stateful* resolveu que o filtro de pacotes simples não resolvia?
4. Como as fontes descrevem a ordem de avaliação das regras e por que a ordem importa?
5. Qual a diferença entre filtragem de ingresso e de egresso, e por que a de egresso costuma ser negligenciada?

**Nível 3 — Crítica e aplicação (e daí?)**
6. Quais limitações do firewall as fontes reconhecem explicitamente?
7. Comparando NIST SP 800-41 e a documentação do pfSense: onde elas concordam e onde a prática moderna diverge da norma de 2009?
8. Monte um checklist de revisão de ruleset usando **apenas** as boas práticas citadas nas fontes.

### 3.2 Variações de prompt testadas

| Versão | Prompt | Resultado obtido | Veredito |
|--------|--------|------------------|----------|
| v1 | `Me explique firewall` | Parágrafo genérico, sem estrutura, sem citações úteis | ❌ Amplo demais |
| v2 | `Quais são os tipos de firewall?` | Lista correta, mas rasa e sem o "porquê" de cada geração | ⚠️ Melhorou |
| v3 | `Liste os tipos de firewall descritos nas fontes, indicando para cada um: camada OSI, o que inspeciona, vantagem principal e limitação. Formate como tabela e cite a fonte de cada linha.` | Tabela estruturada, citações clicáveis, comparação direta | ✅ Prompt final |
| v4 | `Responda em português do Brasil. [prompt v3]` | Mesma qualidade, sem mistura de idiomas | ✅ Versão adotada |

**Padrão que funcionou** (virou meu template):

```
[IDIOMA] + [PAPEL/NÍVEL] + [TAREFA ESPECÍFICA] + [CAMPOS OBRIGATÓRIOS] + [FORMATO] + [CITE AS FONTES]
```

Exemplo aplicado:

> *Responda em português do Brasil, para alguém de nível intermediário em redes. Compare firewall stateless e stateful segundo as fontes, cobrindo: (a) o que cada um inspeciona, (b) o que é tabela de estados, (c) que ataque o stateless não detecta. Use tabela comparativa e cite a fonte de cada afirmação.*

### 3.3 Cicatrizes (troubleshooting real)

> 🩹 Registro honesto do que deu errado e como contornei.

**Cicatriz 1 — "Não encontrei isso nas fontes"**
Perguntei sobre *security groups* em nuvem e o NotebookLM respondeu que não havia informação. **Causa:** ele é *grounded* — só responde com base no que foi enviado, e nenhuma fonte cobria nuvem.
**Solução:** ou adicionar fonte nova, ou aceitar o limite e documentá-lo. Escolhi documentar, porque nuvem estava fora do escopo definido.
**Aprendizado:** essa "limitação" é na verdade a maior **qualidade** da ferramenta — ela não alucina para preencher lacuna. A lacuna é da curadoria, não da IA.

**Cicatriz 2 — Resposta em inglês**
Com 4 de 5 fontes em inglês, as respostas vinham misturando idiomas ou em inglês puro.
**Solução:** iniciar todo prompt com `Responda em português do Brasil` e, para termos técnicos, `mantenha o termo original em inglês entre parênteses`.

**Cicatriz 3 — Resposta genérica com fonte longa**
O NIST SP 800-41 tem dezenas de páginas; perguntas amplas geravam respostas que pareciam um resumo de orelha de livro.
**Solução:** ancorar o prompt na estrutura do documento — `com base na seção sobre tecnologias de firewall do NIST SP 800-41...` — e **sempre clicar nas citações** para verificar se o trecho realmente sustenta a afirmação.

**Cicatriz 4 — Conflito silencioso entre fontes**
O NIST usa *screened subnet*; a documentação do pfSense e o uso de mercado falam *DMZ*. A IA misturou os termos sem sinalizar a diferença.
**Solução:** perguntar explicitamente pela divergência — `As fontes usam termos diferentes para o mesmo conceito? Liste as equivalências.` A IA não aponta contradição se você não pedir.

**Cicatriz 5 — Pedido de comando que a fonte não tinha**
Pedi exemplos de regra e recebi resposta vaga, porque o NIST é normativo, não operacional.
**Solução:** direcionar a pergunta para a fonte certa — `Com base no Packet Filtering HOWTO, mostre a sintaxe de uma regra iptables que aceite conexões estabelecidas.`
**Aprendizado:** a pergunta precisa ir ao **documento que tem a resposta**, não ao acervo inteiro.

**Cicatriz 6 — Nota gerada ≠ nota revisada**
Salvei várias respostas como Nota sem revisar. Na hora de consolidar o miniguia, havia repetição e um trecho impreciso sobre NAT.
**Solução:** adotar o ciclo `perguntar → verificar citação → editar com minhas palavras → salvar`. A IA produz insumo; a curadoria é minha.

---

## 4. Miniguia de Estudo

### 4.1 Resumos estruturados

#### Módulo 1 — Fundamentos

Firewall é um dispositivo ou software que **controla o fluxo de tráfego entre redes ou hosts com posturas de segurança diferentes**. Ele não é um produto isolado, e sim a materialização de uma **política de segurança** — se a política não existe no papel, o ruleset vira improviso.

Três princípios sustentam tudo:

- **Default deny (negar por padrão):** tudo é bloqueado, exceto o que foi explicitamente liberado. O oposto (*default allow*) exige que você preveja todo ataque possível — impossível.
- **Menor privilégio:** libere o mínimo necessário (host específico, porta específica, protocolo específico), nunca `any → any`.
- **Defesa em profundidade:** firewall é uma camada. Não substitui patching, antivírus, IDS/IPS, autenticação forte ou backup.

Posicionamento possível: **perímetro** (borda da rede), **segmentação interna** (entre setores/VLANs) e **host-based** (no próprio servidor ou estação).

#### Módulo 2 — Gerações e tipos

| Tipo | Camada OSI | O que inspeciona | Ponto forte | Limitação |
|------|-----------|------------------|-------------|-----------|
| **Filtro de pacotes (stateless)** | 3 e 4 | IP origem/destino, porta, protocolo, flags | Rápido, baixo overhead | Não tem memória: avalia cada pacote isoladamente |
| **Stateful inspection** | 3 e 4 (+ contexto) | O mesmo + estado da conexão | Libera retorno automaticamente; detecta pacote fora de contexto | Consome memória (tabela de estados); alvo de exaustão |
| **Proxy / Application gateway** | 7 | Conteúdo do protocolo (HTTP, FTP, SMTP) | Isola cliente e servidor; inspeção profunda | Lento; precisa de proxy por protocolo |
| **NGFW** | 3 a 7 | Aplicação, usuário, conteúdo, assinaturas | Identifica app independente da porta; IPS integrado | Custo, complexidade, inspeção TLS é polêmica |
| **WAF** | 7 (web) | Requisições HTTP/HTTPS | Protege contra SQLi, XSS e afins (OWASP Top 10) | Só protege aplicação web; exige tuning contra falso positivo |
| **Host-based / pessoal** | 3 a 7 | Tráfego do próprio host | Protege contra ameaça lateral interna | Gerenciamento distribuído; pode ser desativado pelo usuário |

**O salto conceitual mais importante:** o firewall *stateless* precisa de uma regra manual para liberar o tráfego de retorno — o que costuma resultar em regras amplas e perigosas. O *stateful* mantém uma **tabela de estados**: ao permitir o primeiro pacote de uma conexão, ele cria uma entrada e automaticamente permite os pacotes relacionados de volta. Foi isso que tornou rulesets restritivos viáveis na prática.

#### Módulo 3 — Arquiteturas

- **Roteador de triagem (screening router):** ACL no roteador de borda. Barato, protege pouco.
- **Dual-homed host:** host com duas interfaces, sem roteamento direto. Todo tráfego passa por ele.
- **DMZ / screened subnet:** sub-rede intermediária para serviços públicos (web, e-mail, DNS). Se o servidor público for comprometido, o atacante **não cai direto na LAN**. É o conceito mais cobrado em prova e o mais mal implementado na prática.
- **Firewalls em camadas:** perímetro + interno, idealmente de fabricantes diferentes, para que uma vulnerabilidade não derrube as duas barreiras.
- **Segmentação interna / microsegmentação:** zonas de confiança dentro da rede. Evita que um ransomware que entrou pelo RH alcance o servidor financeiro.

> **Regra mental:** desenhe as **zonas de confiança** antes de escrever qualquer regra. O ruleset é consequência da topologia, não o contrário.

#### Módulo 4 — Anatomia e lógica das regras

Uma regra tipicamente define:

`AÇÃO` (pass / block / reject) · `INTERFACE` · `DIREÇÃO` · `PROTOCOLO` · `ORIGEM (IP/rede/porta)` · `DESTINO (IP/rede/porta)` · `LOG` · `DESCRIÇÃO`

**Quatro pontos que separam iniciante de profissional:**

1. **Ordem importa — *first match wins*.** A avaliação é sequencial, de cima para baixo. Uma regra permissiva no topo anula (*shadowing*) toda restrição abaixo dela.
2. **Block vs Reject.** *Block* descarta silenciosamente (o cliente sofre timeout, e o atacante não sabe se o host existe). *Reject* devolve uma mensagem de recusa (falha rápida, melhor experiência interna). Convenção usual: **reject na LAN, block na WAN**.
3. **Ingress ≠ Egress.** Filtrar entrada é o óbvio. Filtrar **saída** é o que impede exfiltração de dados, comunicação com C2 de malware e uso indevido de serviços. É onde quase toda rede é permissiva demais.
4. **Descrição obrigatória.** Regra sem descrição vira regra órfã: ninguém remove por medo de quebrar algo. Ruleset sem documentação envelhece mal.

**Exemplo em iptables (política restritiva mínima):**

```bash
# Política padrão: nega tudo
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Loopback é confiável
iptables -A INPUT -i lo -j ACCEPT

# Permite retorno de conexões já estabelecidas (o "pulo do gato" stateful)
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Libera SSH apenas da rede de administração
iptables -A INPUT -p tcp -s 192.168.10.0/24 --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# Registra o que foi descartado (para auditoria)
iptables -A INPUT -m limit --limit 5/min -j LOG --log-prefix "FW-DROP: "
```

#### Módulo 5 — Operação e ciclo de vida

O firewall não termina no deploy:

- **Log e monitoramento:** sem log, você descobre o incidente pelo cliente reclamando.
- **Revisão periódica do ruleset:** caçar regras órfãs, duplicadas, sombreadas e temporárias que viraram permanentes.
- **Gestão de mudanças:** toda regra deve ter solicitante, justificativa, data e, quando aplicável, prazo de validade.
- **Testes:** validar o comportamento esperado com varredura (`nmap`) a partir de fora e de dentro.
- **Patching do próprio firewall:** ele é um software exposto — e portanto um alvo.

#### Módulo 6 — Limitações (o que o firewall NÃO faz)

- Não inspeciona o que **não passa por ele** (VPN do usuário, 4G, pen drive, Wi-Fi paralelo).
- Não enxerga bem tráfego **criptografado** sem inspeção TLS — e ela traz custo e implicações de privacidade.
- Não impede **ameaça interna** nem usuário legítimo agindo mal.
- Não protege contra **engenharia social**: o phishing entra pela porta 443, que está liberada.
- Não corrige **aplicação vulnerável**: um SQL Injection trafega por HTTPS perfeitamente válido.
- **A maior causa de falha não é técnica, é de configuração.** Regra mal escrita derruba mais empresa que zero-day.

---

### 4.2 Glossário

| Termo | Definição |
|-------|-----------|
| **ACL (Access Control List)** | Lista ordenada de regras que define qual tráfego é permitido ou negado |
| **Alias / Object group** | Apelido para um conjunto de IPs, redes ou portas, usado para simplificar o ruleset |
| **Bastion host** | Host endurecido e exposto deliberadamente, projetado para resistir a ataque |
| **Chain (netfilter)** | Sequência de regras associada a um ponto do fluxo de pacotes (INPUT, OUTPUT, FORWARD) |
| **Conntrack** | Módulo do Linux que rastreia o estado das conexões |
| **Default deny** | Política em que tudo é negado exceto o explicitamente permitido |
| **Defesa em profundidade** | Estratégia de múltiplas camadas independentes de proteção |
| **DMZ (zona desmilitarizada)** | Sub-rede isolada onde ficam os serviços acessíveis pela internet |
| **DPI (Deep Packet Inspection)** | Inspeção do conteúdo do pacote, não só do cabeçalho |
| **Egress filtering** | Filtragem do tráfego que **sai** da rede |
| **Estado da conexão** | Classificação do pacote: NEW, ESTABLISHED, RELATED, INVALID |
| **Firewall pessoal / host-based** | Firewall executado no próprio dispositivo (ufw, firewalld, Windows Defender Firewall) |
| **Ingress filtering** | Filtragem do tráfego que **entra** na rede |
| **IPS (Intrusion Prevention System)** | Sistema que detecta e bloqueia ataques por assinatura ou comportamento |
| **Menor privilégio** | Conceder apenas o acesso estritamente necessário |
| **NAT (Network Address Translation)** | Tradução de endereços. **Não é mecanismo de segurança**, embora obscureça a rede interna |
| **NGFW (Next-Generation Firewall)** | Firewall com reconhecimento de aplicação, identidade de usuário e IPS integrado |
| **nftables** | Framework de filtragem do Linux, sucessor do iptables |
| **Packet filter** | Firewall que decide com base em cabeçalhos de camada 3 e 4 |
| **Política de firewall** | Documento que define o que deve ser permitido — antecede o ruleset |
| **Proxy / Application gateway** | Intermediário que termina a conexão e a recria, inspecionando camada 7 |
| **Regra órfã** | Regra cuja justificativa original se perdeu e que ninguém ousa remover |
| **Reject vs Block (Drop)** | Recusar avisando o remetente vs descartar silenciosamente |
| **Ruleset** | Conjunto completo de regras ativas no firewall |
| **Segmentação de rede** | Divisão da rede em zonas com regras de trânsito entre elas |
| **Shadowed rule** | Regra que nunca é aplicada porque outra acima dela já casa com o tráfego |
| **Stateful inspection** | Filtragem que considera o contexto da conexão via tabela de estados |
| **Stateless** | Filtragem que avalia cada pacote isoladamente, sem memória |
| **Tabela de estados** | Estrutura que armazena as conexões ativas conhecidas pelo firewall |
| **WAF (Web Application Firewall)** | Firewall especializado em proteger aplicações web na camada 7 |
| **Zero Trust** | Modelo que não assume confiança por localização de rede; tudo é verificado |

---

### 4.3 Prompts reutilizáveis

Prompts testados e prontos para revisões futuras. Substitua `[TEMA]` pelo assunto desejado — funcionam para qualquer caderno temático.

#### 🧭 Mapeamento inicial
```
Responda em português do Brasil. Com base exclusivamente nas fontes deste caderno,
liste os principais conceitos de [TEMA] em ordem de pré-requisito (do que preciso
entender primeiro ao que depende dos anteriores). Para cada conceito, escreva uma
frase de definição e cite a fonte.
```

#### 📊 Comparação estruturada
```
Responda em português do Brasil. Compare [CONCEITO A] e [CONCEITO B] segundo as
fontes, em formato de tabela com as colunas: definição, quando usar, vantagem,
limitação e fonte. Não use informação externa às fontes.
```

#### 🔍 Aprofundamento dirigido
```
Com base na seção sobre [ASSUNTO] da fonte [NOME DA FONTE], explique [CONCEITO]
para alguém de nível intermediário. Mantenha os termos técnicos em inglês entre
parênteses. Finalize com uma analogia prática do dia a dia de TI.
```

#### ⚖️ Detecção de divergência entre fontes
```
As fontes deste caderno divergem, usam terminologias diferentes ou se contradizem
sobre [TEMA]? Liste cada divergência indicando qual fonte diz o quê. Se não houver
divergência, diga explicitamente que não encontrou.
```

#### 🕳️ Mapeamento de lacunas
```
Quais aspectos importantes de [TEMA] NÃO são cobertos pelas fontes deste caderno?
Liste as lacunas e sugira que tipo de fonte adicional as preencheria.
```

#### ✅ Checklist operacional
```
Monte um checklist prático de [TAREFA] usando apenas as boas práticas citadas nas
fontes. Para cada item: o que verificar, por que importa e a fonte. Máximo 12 itens.
```

#### 🎓 Autoavaliação
```
Crie 10 questões de múltipla escolha sobre [TEMA] com base nas fontes, em nível
intermediário. Inclua distratores plausíveis. Apresente primeiro só as questões;
depois, em seção separada, o gabarito comentado com a fonte de cada resposta.
```

#### 🗣️ Teste de compreensão ativa (Feynman)
```
Vou explicar [CONCEITO] com minhas palavras e você avalia com base nas fontes:
"[SUA EXPLICAÇÃO AQUI]".
Aponte: (1) o que está correto, (2) o que está impreciso, (3) o que faltou.
Cite a fonte em cada correção.
```

#### 🧠 Revisão espaçada
```
Gere 15 flashcards sobre [TEMA] no formato "Pergunta | Resposta", uma linha por
card, separados por pipe, prontos para importar no Anki. Baseie-se só nas fontes.
```

#### 🌍 Aplicação em cenário real
```
Considere o cenário: [DESCREVA A SITUAÇÃO].
Usando apenas as recomendações das fontes, indique que abordagem se aplica, quais
são os riscos e o que as fontes NÃO respondem sobre esse cenário.
```

---

## 5. Como reproduzir este caderno

1. Acesse o [NotebookLM](https://notebooklm.google.com) e crie um novo notebook chamado `Firewall — Fundamentos e Prática`.
2. Faça upload das fontes da seção 2 (PDFs) e adicione as fontes web pela opção de link/site.
3. Use os prompts da seção 4.3 seguindo a escada: mapeamento → comparação → aprofundamento → lacunas → autoavaliação.
4. **Verifique cada citação** clicando nos marcadores antes de salvar qualquer resposta como Nota.
5. Consolide as Notas revisadas no miniguia, reescrevendo com suas próprias palavras.
6. Registre nesta seção as suas próprias cicatrizes — o que deu errado é o que mais ensina.

---

## 6. Aprendizados e próximos passos

### O que aprendi sobre estudar com IA

- **A qualidade da resposta é filha da qualidade da fonte.** Curadoria ruim, resposta ruim — a IA não conserta acervo fraco.
- **Prompt amplo gera resposta amplamente inútil.** Escopo + campos obrigatórios + formato + citação mudou tudo.
- **"Não encontrei nas fontes" é uma boa resposta**, não uma falha. É o comportamento *grounded* funcionando.
- **A IA não sinaliza contradição por conta própria** — você precisa perguntar pela divergência.
- **Verificar citação é inegociável.** Ler o trecho original é o que transforma resposta gerada em conhecimento próprio.

### Próximos passos

- [ ] Adicionar fontes sobre firewall em nuvem (security groups, NACLs) para fechar a lacuna identificada
- [ ] Montar um laboratório com pfSense em VM e aplicar o checklist do Módulo 5
- [ ] Reescrever o ruleset de exemplo em `nftables` e comparar com a versão `iptables`
- [ ] Repetir a metodologia para o tema seguinte: **Zero Trust**

---

## 📄 Licença

Material de estudo pessoal. As fontes citadas pertencem aos seus respectivos autores e mantêm suas licenças originais.

---

<div align="center">

**Desenvolvido como parte do desafio de projeto da [DIO](https://www.dio.me)** 🚀

</div>
