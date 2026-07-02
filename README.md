# Guia de Diagnóstico: Problemas Comuns em VPN e Acesso Remoto

**Autor:** Davi Senise
**Cargo:** Técnico de Suporte TI (N1/N2)
**Repositório:** vpn-remote-access-guide

---

## Objetivo

Referência rápida pros problemas mais recorrentes de VPN e acesso remoto em ambiente corporativo. Cada seção segue: **Sintoma > Causa provável > Como confirmar > Solução > Como evitar de novo.**

---

## Índice

1. [VPN não conecta](#1-vpn-não-conecta)
2. [VPN conecta mas sem acesso aos recursos internos](#2-vpn-conecta-mas-sem-acesso-aos-recursos-internos)
3. [MFA falha na hora de autenticar](#3-mfa-falha-na-hora-de-autenticar)
4. [VPN cai sozinha durante o uso](#4-vpn-cai-sozinha-durante-o-uso)
5. [Lentidão extrema usando VPN](#5-lentidão-extrema-usando-vpn)
6. [Certificado expirado ou inválido](#6-certificado-expirado-ou-inválido)
7. [Checklist rápido de coleta de dados](#7-checklist-rápido-de-coleta-de-dados)

---

## 1. VPN não conecta

**Sintoma:** Cliente de VPN dá erro ao tentar conectar, ou fica travado em "Conectando..." indefinidamente.

**Causas prováveis:**
- Credenciais expiradas ou senha alterada recentemente sem atualizar no cliente
- Servidor de VPN fora do ar ou em manutenção
- Firewall local (do usuário, hotel, rede pública) bloqueando a porta usada pela VPN

**Como confirmar:**
Testa em outra rede (dado móvel do celular, por exemplo). Se conectar lá, o problema é a rede de origem bloqueando a porta, não o cliente ou o servidor.

**Solução:**
- Confirma se a senha da conta ainda é válida (login duplo em outro sistema corporativo)
- Testa em rede diferente pra isolar bloqueio de firewall local
- Reinstala ou atualiza o cliente de VPN se o erro for genérico e recorrente

**Como evitar de novo:**
Orientar usuários que trabalham remoto a evitar redes públicas com bloqueio agressivo de porta, e manter o cliente de VPN sempre atualizado.

---

## 2. VPN conecta mas sem acesso aos recursos internos

**Sintoma:** VPN mostra status "Conectado", mas o usuário não consegue acessar sistemas, pastas de rede ou intranet.

**Causas prováveis:**
- Rota de rede (split tunneling) configurada errado, direcionando tráfego pro lugar errado
- DNS interno não sendo usado durante a conexão VPN
- Permissão de acesso ao recurso específico não concedida pro usuário

**Como confirmar:**
```
ipconfig /all
```
Verifica se o DNS que aparece é o servidor DNS interno da empresa, não um DNS público. Se for público, a resolução de nomes internos vai falhar mesmo com VPN ativa.

**Solução:**
- Ajusta configuração de DNS no perfil de VPN pra usar o DNS interno
- Testa acesso por IP direto ao recurso (em vez de nome) pra confirmar se é problema de DNS ou de permissão
- Se por IP também falhar, é permissão, não conectividade, escala pro time responsável pelo recurso

**Como evitar de novo:**
Documentar o perfil de configuração padrão de VPN (DNS, rotas, split tunneling) evita reconfiguração manual repetida a cada usuário novo.

---

## 3. MFA falha na hora de autenticar

**Sintoma:** VPN pede segundo fator de autenticação e ele falha, expira, ou nunca chega (push notification, SMS, código).

**Causas prováveis:**
- Dessincronização de horário no dispositivo (afeta apps de autenticação baseados em tempo)
- Aplicativo autenticador desatualizado ou com conta desvinculada
- Sinal de celular fraco impedindo recebimento de SMS/push

**Como confirmar:**
Verifica a hora do dispositivo do usuário comparada ao horário real. Diferença de poucos minutos já quebra a maioria dos tokens TOTP.

**Solução:**
- Corrige a sincronização automática de data/hora no dispositivo
- Reenvia o código ou tenta método alternativo (SMS em vez de push, por exemplo)
- Se o app autenticador estiver com problema, revincula a conta seguindo o procedimento de reset de MFA da empresa

**Como evitar de novo:**
Padronizar sincronização automática de horário como parte do checklist de configuração de qualquer dispositivo novo.

---

## 4. VPN cai sozinha durante o uso

**Sintoma:** Conexão VPN funciona no início mas desconecta sozinha depois de um tempo, exigindo reconexão manual.

**Causas prováveis:**
- Instabilidade na rede local do usuário (Wi-Fi fraco, quedas intermitentes)
- Timeout de sessão configurado no servidor de VPN
- Computador entrando em modo de suspensão/economia de energia e derrubando a interface de rede

**Como confirmar:**
```
ping -t 8.8.8.8
```
Roda em paralelo enquanto usa a VPN. Se o ping cair nos mesmos momentos que a VPN desconecta, o problema é da rede local, não da VPN em si.

**Solução:**
- Rede instável: resolve na origem (ver guia de rede/modem)
- Timeout de sessão: ajustar política no servidor, se fizer sentido pro perfil de uso
- Suspensão: desativa economia de energia agressiva pra adaptador de rede em `Gerenciador de Dispositivos > Adaptador de Rede > Propriedades > Gerenciamento de Energia`

**Como evitar de novo:**
Orientar usuários remotos a evitar Wi-Fi instável durante tarefas críticas e preferir cabo quando possível.

---

## 5. Lentidão extrema usando VPN

**Sintoma:** Navegação e acesso a sistemas internos ficam muito mais lentos com VPN ativa do que sem ela.

**Causas prováveis:**
- Todo o tráfego (inclusive navegação geral de internet) passando pelo túnel VPN sem necessidade (full tunneling mal configurado)
- Servidor de VPN sobrecarregado por excesso de conexões simultâneas
- Distância geográfica grande até o servidor de VPN

**Como confirmar:**
Compara velocidade de navegação comum (sites externos, não corporativos) com e sem VPN ativa. Se a diferença for grande só em sites externos, é configuração de tunneling.

**Solução:**
- Configura split tunneling: só tráfego destinado à rede interna passa pela VPN, o resto vai direto
- Se o servidor estiver sobrecarregado, é decisão de infraestrutura (balanceamento, mais capacidade), escalar pra quem administra

**Como evitar de novo:**
Revisar periodicamente a configuração de tunneling conforme o volume de usuários remotos cresce.

---

## 6. Certificado expirado ou inválido

**Sintoma:** Erro mencionando certificado, "conexão não confiável", ou falha de autenticação relacionada a certificado digital.

**Causas prováveis:**
- Certificado do cliente expirado
- Certificado do servidor renovado sem atualizar a referência no cliente
- Relógio do sistema desincronizado (certificados são sensíveis a data/hora)

**Como confirmar:**
Verifica a validade do certificado instalado (via gerenciador de certificados do Windows, `certmgr.msc`) e compara com a data atual.

**Solução:**
- Renova ou reemite o certificado conforme o procedimento da empresa
- Corrige data/hora do sistema se estiver incorreta
- Reinstala o perfil de VPN completo se o certificado antigo estiver corrompido

**Como evitar de novo:**
Monitorar data de expiração de certificados com antecedência (calendário ou alerta automatizado) evita que o problema apareça de surpresa pro usuário final.

---

## 7. Checklist rápido de coleta de dados

- [ ] Erro exato mostrado pelo cliente de VPN?
- [ ] Funciona em outra rede (ex: dado móvel)?
- [ ] Data/hora do dispositivo está correta?
- [ ] DNS usado durante a conexão é o interno da empresa?
- [ ] Problema acontece com um usuário só ou vários (indica servidor)?
- [ ] Certificado (se aplicável) está dentro da validade?

---

## Notas

Guia vivo. Adicionar novas seções conforme padrões de falha aparecerem em atendimentos reais, seguindo a mesma estrutura.
