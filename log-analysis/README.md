# 🔵 Análise de Logs de Autenticação SSH

![Status](https://img.shields.io/badge/status-completo-brightgreen)
![Nível](https://img.shields.io/badge/nível-iniciante-blue)
![Área](https://img.shields.io/badge/área-blue%20team-informational)

## 📌 Visão Geral
Projeto prático de investigação de logs de autenticação em um servidor
SSH local, replicando o fluxo real de triagem de um analista de SOC
ao identificar tentativas de acesso falhas e bem-sucedidas.

## 🎯 Objetivo
Configurar um serviço de autenticação, gerar eventos de login controlados,
extrair esses eventos dos logs nativos do sistema (journalctl) e
interpretar o que cada tipo de evento significa em um cenário real.

## 🧠 Conceitos Envolvidos

**O que é o log de autenticação:** todo sistema Linux moderno registra
eventos de autenticação através do systemd/journald — um serviço central
que recebe mensagens de outros processos (como o SSH) e as armazena de
forma consultável. É o equivalente Linux ao log de Segurança do Windows.

**O que é PAM:** o Pluggable Authentication Modules é o componente que
efetivamente verifica usuário e senha no Linux — é ele quem decide se a
autenticação passa ou falha, e quem reporta o resultado para o log.

**Diferença entre tipos de falha:** o log distingue duas situações
diferentes, que representam riscos diferentes:
- `Invalid user`: alguém tentou logar com um usuário que **não existe**
  no sistema — indica reconhecimento (tentativa de descobrir contas válidas).
- `Failed password`: o usuário **existe**, mas a senha estava errada —
  indica tentativa de quebra de senha contra uma conta real, um risco maior.

## 🛠️ Ferramentas e Tecnologias
- Linux (Debian via Chromebook Linux/Crostini)
- OpenSSH Server
- journalctl, grep (linha de comando)

## 📋 Metodologia

1. **Instalação e ativação do servidor SSH**, para ter um serviço real
   gerando logs de autenticação (`sudo apt install openssh-server`).
2. **Ajuste de configuração** para permitir login por senha
   (`PasswordAuthentication yes` em `/etc/ssh/sshd_config`), já que por
   padrão o SSH prioriza autenticação por chave.
3. **Geração controlada de eventos de teste**: tentativas de login com
   senha errada de propósito, seguidas de uma tentativa com a senha correta.
4. **Extração dos eventos** relevantes do log do sistema, filtrando
   ruído com `journalctl -u ssh` e `grep`.
5. **Interpretação do padrão temporal** encontrado, comparando com o
   que caracterizaria um ataque de força bruta real.

## 📊 Resultados

| Horário  | Evento                    | Usuário |
|----------|---------------------------|---------|
| 22:33:28 | authentication failure    | angel   |
| 22:33:30 | Failed password            | angel   |
| 22:33:36 | Failed password            | angel   |
| 22:33:42 | **Accepted password**      | angel   |

## 🔍 Análise
Foram observadas 2 tentativas de senha incorreta seguidas de um login
bem-sucedido, em um intervalo de 14 segundos — um padrão consistente
com erro humano legítimo (poucas tentativas, sucesso rápido), diferente
de um ataque de força bruta real, que tipicamente gera dezenas de
tentativas em segundos, de forma automatizada.

Durante o processo, também foram observadas tentativas com o evento
`Invalid user`, geradas ao testar um nome de usuário com grafia
diferente (maiúsculas) do usuário real do sistema — evidenciando na                      
prática a diferença entre os dois tipos de evento descritos acima.
## 🚨 O que justificaria um alerta
Com base no padrão observado, a regra de detecção proposta seria:

"IF count("Failed password", mesmo usuário) >= 5 IN 60s
THEN gerar alerta de possível força bruta"

(o número de tentativas do teste, 2, fica abaixo desse limiar —
por isso não caracteriza um ataque, e sim uso legítimo com erro humano)

## 🧠 Competências Demonstradas
- Configuração de serviço de autenticação (SSH) em ambiente Linux
- Leitura e interpretação de logs via journalctl
- Diferenciação entre tipos de evento de autenticação e seus riscos
- Raciocínio de detecção: diferenciar erro humano de padrão de ataque
- Documentação técnica no formato usado por times de SOC reais

## 📚 Aprendizados
Entendi como o systemd/journald registra eventos de autenticação, a
diferença prática entre falha por usuário inválido e por senha
incorreta, e como isolar eventos relevantes em meio ao ruído do log
usando journalctl e grep.

## 🔁 Como reproduzir este projeto
1. `sudo apt install -y openssh-server`
2. `sudo service ssh start`
3. Editar `/etc/ssh/sshd_config` → `PasswordAuthentication yes`
4. `sudo service ssh restart`
5. `ssh SEU_USUARIO@localhost` (testar senha errada e depois certa)
6. `sudo journalctl -u ssh --no-pager | grep -i "SEU_USUARIO"`

---
📎 Parte da minha trilha de transição para Cibersegurança —
veja os outros projetos no repositório principal.

## 🚨 O que justificaria um alerta
Com base no padrão observado, a regra de detecção proposta seria:
