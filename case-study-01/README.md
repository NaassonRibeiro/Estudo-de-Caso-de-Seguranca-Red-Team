# Estudo de Caso 01: Avaliação de Vulnerabilidade em Aplicação Web e Infraestrutura Externa

## 1. Objetivo e Escopo

O objetivo desta análise autorizada era identificar vulnerabilidades na aplicação web principal de uma empresa corporativa. O escopo permitia uma avaliação "black box", com a restrição de evitar ações intrusivas que pudessem impactar a performance da rede ou disparar alertas de segurança (WAF/Firewall).

## 2. Metodologia: Uma Abordagem Investigativa

Devido às proteções existentes (Cloudflare, Firewall), uma abordagem manual e focada em inteligência foi adotada em vez de scanners automatizados.

### Fase 1: Reconhecimento e Hipótese Inicial (Pivô)
1.  **Análise Passiva:** Utilizando extensões de navegador (ex: Wappalyzer), identifiquei o stack tecnológico básico da aplicação.
2.  **Pivô Estratégico:** A aplicação parecia robusta em testes superficiais. No rodapé, identifiquei a empresa desenvolvedora do site. **Minha hipótese foi que o desenvolvedor poderia reutilizar templates de projetos antigos e que sua própria infraestrutura poderia ser menos segura**, revelando informações sobre as tecnologias e práticas de desenvolvimento encobertas por camadas de proteção.
3.  **Coleta de Inteligência:** A análise passiva do portfólio e do site do desenvolvedor confirmou o uso de um stack tecnológico específico e sugeriu uma política de atualizações inconsistente. Com essa informação, voltei ao alvo original com uma suspeita mais forte de possíveis vulnerabilidades.

### Fase 2: Exploração Manual da Aplicação Web
1.  **Motivação e Foco:** **Motivado pelos indicativos de tecnologias defasadas encontrados no site do desenvolvedor (Fase 1), meu foco se voltou para encontrar uma falha humana ou de configuração que pudesse ser usada para contornar as proteções da aplicação principal.**
2.  **Identificação da Anomalia:** Aprofundei a análise no tratamento das rotas `HTTP` e `HTTPS`. Minha hipótese era que a lógica de redirecionamento poderia conter inconsistências que poderiam ser exploradas.
3.  **Técnica de Manipulação de Roteamento:** Forcei algumas requisições `GET` diretamente via `HTTP` em cenários incomuns para um endpoint que esperava ser acessado apenas por `HTTPS`, buscando uma falha de lógica no tratamento dessas exceções.
4.  **Resultado:** A aplicação entrou em colapso (`crash`), contornando o tratamento de erros padrão e expondo a página de debug nativa do PHP.

### Fase 3: OSINT e Análise da Infraestrutura Externa
1.  **Footprinting Inicial:** Através do `WHOIS` do domínio, obtive o AS Number (ASN), mesmo que este apontasse para a Cloudflare.
2.  **Mapeamento da Rede:** Utilizando ferramentas online de análise de BGP, mapeei o ASN e calculei os ranges de IP que realmente pertenciam à organização.
3.  **Descoberta de Infraestrutura Legada:** A investigação revelou traços de roteamento da infraestrutura antiga, pré-migração para a Cloudflare. Verifiquei os ranges de IP associados a esses traços.
4.  **Verificação Passiva:** Com os IPs em mãos, usei ferramentas online de `ping` e port scan (através de VPN para OpSec) para identificar hosts ativos e serviços expostos.

## 3. Descobertas (Findings)

### Finding 1.1: (CRÍTICO) Exposição de Informações Sensíveis por Tratamento de Erro Inadequado
* **Descrição:** A exploração de uma falha na lógica de roteamento `HTTP/HTTPS` levou à exposição de versões de software (PHP, MySQL, Laravel), caminhos de diretórios no servidor e trechos de código-fonte.
* **Impacto:** As versões expostas possuíam **CVEs conhecidas para Execução Remota de Código (RCE)**. Um atacante poderia usar essas informações para obter controle total do servidor. A proteção da Cloudflare mascarava a vulnerabilidade, mas não a eliminava.
* **Evidência:**
    ![Prova de Conceito Sanitizada - Erro na Aplicação Web](./evidence/webapp-exposto.png)

### Finding 1.2: (ALTO) Exposição de Infraestrutura de Servidores Legados
* **Descrição:** Servidores da infraestrutura antiga da empresa foram encontrados ativos e acessíveis publicamente na internet.
* **Impacto:** Scans de portas revelaram serviços desatualizados com vulnerabilidades de criticidade média a alta. Este servidor representava um ponto de entrada fácil para a rede corporativa e poderia ser usado para lançar ataques internos ou ser comprometido para outros fins (ex: botnet).
* **Evidência:**
    ![Prova de Conceito Sanitizada - Scan em Servidor Legado](./evidence/scan_servidor.png)

## 4. Conclusão da Análise 1

A investigação demonstrou que, mesmo por trás de camadas de proteção como WAFs, vulnerabilidades críticas podem existir na aplicação e na infraestrutura. A abordagem combinada de análise manual da aplicação e OSINT profundo foi essencial para descobrir riscos que scanners automatizados por sua estrutura de técnicas convencionais não identificariam. O relatório completo foi entregue à equipe responsável.
