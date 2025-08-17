# Estudo de Caso 01: Avaliação de Vulnerabilidade em Aplicação Web e Infraestrutura Externa

## 1. Objetivo e Escopo
O objetivo desta análise autorizada era identificar vulnerabilidades na aplicação web principal de uma empresa corporativa. O escopo permitia uma avaliação "black box", com a restrição de evitar ações intrusivas que pudessem impactar a performance da rede ou disparar alertas de segurança (WAF/Firewall).

## 2. Metodologia: Uma Abordagem Investigativa
Devido às proteções existentes (Cloudflare, Firewall), uma abordagem manual e focada em inteligência foi adotada em vez de scanners automatizados.

### Fase 1: Reconhecimento e Hipótese Inicial (Pivô)
A análise passiva do alvo inicial e de seu desenvolvedor sugeriu o uso de um stack tecnológico com uma política de atualizações inconsistente. Com essa informação, a investigação retornou ao alvo original com uma suspeita mais forte de possíveis vulnerabilidades de configuração ou software desatualizado.

### Fase 2: Exploração Manual da Aplicação Web
1.  **Motivação e Foco:** Motivado pelos indicativos da Fase 1, meu foco se voltou para encontrar uma falha humana ou de configuração que pudesse ser usada para contornar as proteções da aplicação.
2.  **Identificação da Anomalia:** Aprofundei a análise no tratamento das rotas `HTTP` e `HTTPS`, buscando inconsistências na lógica de redirecionamento.
3.  **Técnica de Manipulação de Roteamento:** Forcei requisições `GET` diretamente via `HTTP` em cenários incomuns para um endpoint que esperava ser acessado apenas por `HTTPS`.
4.  **Resultado:** A aplicação entrou em colapso (`crash`), contornando o tratamento de erros padrão e expondo a página de debug nativa do PHP.

### Fase 3: OSINT e Análise da Infraestrutura Externa
A investigação da infraestrutura foi realizada a partir do `WHOIS` e do mapeamento do AS Number (ASN), que revelou traços de roteamento de servidores legados. Com os IPs em mãos, usei ferramentas online de `ping` e port scan. Para garantir o anonimato e evitar a correlação com a rede da empresa, toda a análise foi feita através de VPN e com a alteração do User-Agent do navegador para simular um sistema operacional diferente, dificultando a identificação nos logs do servidor.

## 3. Descobertas (Findings)

### Finding 1.1: (CRÍTICO) Exposição de Informações Sensíveis por Tratamento de Erro Inadequado
* **Descrição:** A vulnerabilidade foi acionada ao forçar uma requisição `GET` em uma rota que suportava apenas o método `POST`. Isso expôs a página de debug nativa do PHP, revelando:
    * **Versões de Software:** PHP 7.3.33, Laravel 7.23.0, Symfony, Illuminate e MySQL.
    * **Path Disclosure:** Caminhos completos de arquivos no servidor e a linha exata do erro (ex: `AbstractRouteCollection.php:117`).
    * **Estrutura Interna:** Trechos do código-fonte que revelavam a lógica da aplicação e referências a diretórios internos.
* **Impacto:** Um atacante poderia usar as versões de software para encontrar exploits conhecidos (RCE). A exposição da estrutura interna facilitaria o planejamento de ataques mais sofisticados. A proteção da Cloudflare mascarava, mas não eliminava o risco.
* **Evidência:**
    ![Prova de Conceito Sanitizada - Erro na Aplicação Web](./evidence/webapp-vulneravel.png)

### Finding 1.2: (ALTO) Exposição de Infraestrutura de Servidores Legados
* **Descrição:** Servidores da infraestrutura antiga da empresa foram encontrados ativos e acessíveis publicamente na internet.
* **Impacto:** Scans de portas revelaram serviços desatualizados com vulnerabilidades de criticidade média a alta, representando um ponto de entrada fácil para a rede corporativa.
* **Evidência:**
    ![Prova de Conceito Sanitizada - Scan em Servidor Legado](./evidence/Servidor-vulneravel.png)

### Finding 1.3: (MÉDIO) Servidor DNS com Serviço Vulnerável Exposto
* **Descrição:** Durante a fase de OSINT, foi identificado um servidor de DNS da organização executando uma versão do Nginx com uma vulnerabilidade conhecida publicamente (CVE-[código da CVE]).
* **Impacto:** A exploração dessa CVE poderia levar à negação de serviço no DNS, vazamento de informações da infraestrutura ou ser usada como pivô para ataques internos.

## 4. Conclusão da Análise 1
A investigação demonstrou que, mesmo por trás de camadas de proteção, vulnerabilidades críticas podem existir. A abordagem combinada de análise manual da aplicação e OSINT profundo foi essencial para descobrir riscos que scanners automatizados não identificariam.
