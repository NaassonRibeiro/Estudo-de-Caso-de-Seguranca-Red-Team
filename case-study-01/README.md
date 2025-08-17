# Estudo de Caso 01: Bypass de WAF e Descoberta de Infraestrutura Exposta

## 1. Objetivo e Escopo
O objetivo desta análise foi avaliar a postura de segurança externa de uma empresa corporativa através de uma abordagem de duas frentes:
1.  Análise da aplicação web principal, que operava sob a proteção de um WAF da Cloudflare.
2.  Busca por infraestrutura de rede e servidores legados diretamente expostos na internet, fora do perímetro da Cloudflare.

## 2. Metodologia: Uma Abordagem Investigativa
A análise combinou uma exploração manual e criativa da aplicação web com uma fase de OSINT e varredura de rede na infraestrutura externa, evitando o uso de scanners automatizados intrusivos que seriam bloqueados pelo WAF.

### Fase 1: Análise da Aplicação Web (Protegida pela Cloudflare)
A investigação sobre o desenvolvedor da aplicação web, identificado no rodapé do alvo, levantou suspeitas sobre o uso de tecnologias desatualizadas, motivando uma análise mais profunda em busca de falhas de configuração na aplicação principal. A manipulação de rotas `HTTP/HTTPS` em cenários incomuns foi a técnica escolhida para tentar forçar a aplicação a um estado de erro.

### Fase 2: OSINT para Mapeamento da Infraestrutura Real
O objetivo desta fase foi contornar o "escudo" da Cloudflare para encontrar os endereços de IP reais e a infraestrutura legada da empresa. A partir do `WHOIS` e do mapeamento do AS Number (ASN), foi possível identificar ranges de IP históricos e servidores que não estavam sob a proteção do WAF, tornando-os alvos diretos para uma análise mais aprofundada.

## 3. Descobertas (Findings)

### Finding 1.1: (CRÍTICO) Exposição de Versão Vulnerável do PHP Levando a RCE (CVE-2024-4577)
* **Descrição:** Mesmo operando por trás do WAF da Cloudflare, uma falha de lógica na aplicação foi explorada. Ao forçar uma requisição `GET` em uma rota que esperava `POST`, foi possível causar um `crash` e expor a página de debug do PHP, revelando a versão exata em uso: **PHP 7.3.33**.
* **Impacto:** Esta versão do PHP está descontinuada e é comprovadamente vulnerável à **CVE-2024-4577**, uma falha de Injeção de Comando de OS de severidade **Crítica (CVSS 9.3)**. Esta vulnerabilidade, que está sendo ativamente explorada na internet, permite que um atacante remoto e não autenticado execute comandos arbitrários no sistema operacional. Portanto, o vazamento da versão não é apenas um risco teórico; é a **confirmação de um caminho direto para o comprometimento total do servidor (RCE)**, uma falha que o WAF não poderia mitigar.
* **Evidência:**
    ![Prova de Conceito Sanitizada - Erro na Aplicação Web](./evidence/webapp-cloudflare-vulneravel.png)

### Finding 1.2: (ALTO) Infraestrutura Legada Diretamente Exposta com Múltiplas Vulnerabilidades Críticas
* **Descrição:** A investigação de OSINT permitiu a descoberta de servidores legados que **não estavam protegidos pela Cloudflare**. A varredura nestes IPs revelou múltiplos serviços (Apache/IIS, SNMP) significativamente desatualizados e vulneráveis.
* **Destaques das Vulnerabilidades Encontradas:**
  * **Servidores Web:** Foram identificadas múltiplas falhas críticas, incluindo vulnerabilidades de **Path Traversal e Execução Remota de Código** (ex: CVE-2021-41773).
  * **Gerenciamento de Rede (SNMP):** O serviço estava exposto e vulnerável a falhas que poderiam levar ao vazamento de informações da rede interna.
* **Impacto:** O servidor representava um ponto de entrada de baixo esforço e alto impacto. Um atacante poderia obter controle total deste servidor e usá-lo como um pivô para atacar a rede interna, contornando completamente o WAF.
* **Evidência:**
    ![Prova de Conceito Sanitizada - Scan em Servidor Legado](./evidence/scan-infraestrutura-legada.png)

### Finding 1.3: (ALTO) Servidor DNS com Nginx Vulnerável (CVE-2021-23017)
* **Descrição:** Também fora do perímetro da Cloudflare, foi identificado um servidor DNS executando Nginx 1.20.1, vulnerável à CVE-2021-23017.
* **Impacto:** A exploração poderia levar a uma Negação de Serviço (DoS), afetando a resolução de nomes para os serviços da empresa.
    ![Prova de Conceito Sanitizada - Servidor DNS Vulnerável](./evidence/servidor-dns-vulneravel.png)

## 4. Conclusão da Análise 1
Esta análise destaca um erro comum de postura de segurança: a **falsa sensação de segurança** gerada por uma única camada de proteção, como a Cloudflare. Enquanto o WAF mitigava ataques básicos, a aplicação continha falhas de lógica e, mais criticamente, rodava em uma plataforma com vulnerabilidades de RCE conhecidas e ativamente exploradas, e a infraestrutura legada permanecia completamente exposta.
