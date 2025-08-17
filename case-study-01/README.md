# Estudo de Caso 01: Avaliação de Vulnerabilidade em Aplicação Web e Infraestrutura Externa

## 1. Objetivo e Escopo
O objetivo desta análise autorizada era identificar vulnerabilidades na aplicação web principal e na infraestrutura externa de uma empresa corporativa, utilizando uma abordagem black box.

## 2. Metodologia: Uma Abordagem Investigativa
A análise combinou uma exploração manual e criativa da aplicação web com uma fase de OSINT e varredura de rede na infraestrutura externa, evitando o uso de scanners automatizados intrusivos.

### Fase 1: Reconhecimento e Hipótese Inicial (Pivô)
A investigação sobre o desenvolvedor da aplicação web acessível no rodapé do alvo levantou suspeitas sobre o uso de tecnologias desatualizadas, motivando uma análise mais profunda em busca de falhas de configuração.

### Fase 2: Exploração Manual da Aplicação Web
A manipulação de rotas `HTTP/HTTPS` em cenários incomuns forçou a aplicação a um estado de erro, contornando as proteções e expondo a página de debug nativa do PHP.

### Fase 3: OSINT e Análise da Infraestrutura Externa
A partir do `WHOIS` e do mapeamento do ASN, foram identificados e escaneados servidores legados da organização, utilizando ferramentas online e técnicas de OpSec (VPN, User-Agent Spoofing).

## 3. Descobertas (Findings)

### Finding 1.1: (CRÍTICO) Exposição de Informações Sensíveis na Aplicação Web
* **Descrição:** A exploração de uma falha na lógica de roteamento (`GET` vs `POST`) expôs a página de debug do PHP com versões de software (PHP 7.3.33, Laravel 7.23.0), paths de diretórios e trechos de código-fonte.
* **Impacto:** As informações expostas poderiam ser usadas para encontrar exploits conhecidos (RCE) e mapear a aplicação para ataques futuros.
* **Evidência:**
    ![Prova de Conceito Sanitizada - Erro na Aplicação Web](./evidence/webapp-vulneravel.png)

### Finding 1.2: (ALTO) Exposição de Infraestrutura de Servidores Legados com Múltiplas Vulnerabilidades Críticas
* **Descrição:** A varredura de portas no servidor legado revelou múltiplos serviços expostos à internet, incluindo servidores web (Apache/IIS) e de gerenciamento (SNMP). A análise indicou que estes serviços estavam significativamente desatualizados.
* **Destaques das Vulnerabilidades Encontradas:**
  * **Servidores Web (Apache/IIS):** Foram identificadas múltiplas falhas críticas, incluindo vulnerabilidades de **Path Traversal e Execução Remota de Código** (ex: CVE-2021-41773, MS12-073).
  * **Gerenciamento de Rede (SNMP):** O serviço SNMP estava exposto e vulnerável a falhas que poderiam levar ao vazamento de informações detalhadas sobre a rede interna./
* **Impacto:** O servidor representava um ponto de entrada de baixo esforço e alto impacto. Um atacante poderia obter controle total deste servidor e usá-lo como um pivô para atacar a rede interna da empresa.
* **Evidência:**
    ![Prova de Conceito Sanitizada - Scan em Servidor Legado](./evidence/Lista-Vulnerabildiades.png)

### Finding 1.3: (ALTO) Servidor DNS com Nginx Vulnerável a Negação de Serviço (CVE-2021-23017)
* **Descrição:** Foi identificado um servidor DNS executando Nginx 1.20.1, vulnerável à CVE-2021-23017, uma falha no resolver de DNS.
* **Impacto:** A exploração poderia levar a uma Negação de Serviço (DoS), tornando o servidor DNS indisponível e afetando serviços da empresa que dependem dele.
    ![Prova de Conceito Sanitizada - Scan em Servidor Legado](./evidence/Ngnix-vulneravel.png)

## 4. Conclusão da Análise 1
A investigação demonstrou que a confiança em proteções de borda (WAF) pode mascarar falhas críticas internas. A combinação de análise manual e OSINT foi essencial para revelar riscos sistêmicos tanto na aplicação quanto na infraestrutura legada.
