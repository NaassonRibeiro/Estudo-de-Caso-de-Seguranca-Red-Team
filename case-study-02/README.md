# Estudo de Caso 02: Análise de Acompanhamento e Descoberta de Dispositivo de Rede Vulnerável

## 1. Objetivo

Após o primeiro relatório, foi conduzida uma segunda análise para verificar as correções e reavaliar a postura de segurança externa da organização. O objetivo era confirmar se a infraestrutura legada havia sido desativada e procurar por novos pontos de exposição, partindo de uma nova investigação para garantir uma cobertura completa e imparcial.

## 2. Metodologia: OSINT Avançado a Partir do Zero

Adotando a filosofia de que uma nova análise pode revelar o que a anterior não viu, a investigação foi reiniciada do zero, sem utilizar os dados de IP do primeiro relatório como ponto de partida. A estratégia foi conduzir uma varredura ampla na internet em busca de quaisquer ativos digitais associados à empresa.

1.  **Verificação de Remediação:** O primeiro passo foi confirmar que os servidores legados do Estudo de Caso 1 estavam de fato offline, validando a ação da empresa.

2.  **Fase de Reconhecimento Multifacetado (OSINT):** Foram utilizadas diversas técnicas de inteligência de fontes abertas, executadas através de plataformas online para manter a natureza não-intrusiva da análise (nenhum tráfego partiu diretamente da minha máquina).
    * **Google Hacking (Dorking):** Utilizei operadores de busca avançada para encontrar documentos, subdomínios, ou páginas de login publicamente indexadas e associadas ao nome da empresa.
    * **Análise Histórica Digital:** O `Archive.org` foi consultado para revisitar snapshots antigos do site, buscando por nomes de host, tecnologias ou parceiros que pudessem ainda estar em uso na nova infraestrutura.
    * **Varredura em Plataformas de Busca de Dispositivos:** O ponto chave da descoberta veio da utilização de um motor de busca para dispositivos conectados à internet (similar ao Shodan/Censys). Busquei por certificados SSL, nomes de organização e outros artefatos relacionados à empresa em toda a internet.

3.  **A Descoberta (O Pivô):** Foi através de uma dessas plataformas de busca que um **MikroTik** da empresa foi identificado. A plataforma retornou informações cruciais sobre o dispositivo:
    * Seu endereço de IP público.
    * O nome (hostname) do dispositivo, que sugeria sua função interna ("almoxarifado").
    * A versão exata do seu sistema operacional (RouterOS).

4.  **Confirmação e Correlação:** Com o endereço de IP do MikroTik em mãos, realizei uma geolocalização do IP, que bateu com a localização física conhecida da empresa. Isso confirmou que o ativo pertencia à organização e que, apesar de terem mudado sua infraestrutura e ranges de IP, um novo ponto de exposição havia sido criado. A investigação fluiu de um ativo digital encontrado na internet para a confirmação de sua localização e propriedade.

## 3. Descoberta (Finding)

### Finding 2.1: (CRÍTICO) Dispositivo de Rede (MikroTik) Exposto e Vulnerável
* **Descrição:** O roteador MikroTik estava acessível publicamente e executando uma versão de firmware desatualizada.
* **Impacto:** A versão do RouterOS possuía uma **CVE conhecida que permitia bypass de autenticação ou acesso não autorizado**. Um atacante poderia comprometer este roteador para:
    * Realizar ataques de Man-in-the-Middle (MITM) contra o tráfego da empresa.
    * Usá-lo como um pivô para acessar a rede interna.
    * Interromper a conectividade da localidade.
	 ![Prova de Conceito Sanitizada - Mikrotik Vulnerável](/evidence/mikrotik-vulneravel.png)
## 4. Conclusão da Análise 2

Esta segunda análise reforça a importância da **gestão contínua de ativos (Asset Management)** e de uma política de atualizações abrangente. A metodologia, partindo do zero, provou ser eficaz ao descobrir vulnerabilidades em uma infraestrutura completamente nova. A falha demonstra que a segurança não é um projeto único, mas um processo contínuo de vigilância, e que a "sombra de TI" (dispositivos não gerenciados conectados à rede) representa um dos riscos mais significativos para as organizações modernas.
